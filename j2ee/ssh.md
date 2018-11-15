package com.bill99.ctmp.service;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URL;
import java.util.List;
import java.util.Vector;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.log4j.Logger;
import org.apache.log4j.PropertyConfigurator;

import ch.ethz.ssh2.Connection;
import ch.ethz.ssh2.Session;

import com.bill99.ctmp.common.CtmpUtil;
import com.bill99.golden.inf.sso.util.CM;

public class SshService {
	private static Logger log = Logger.getLogger(SshService.class);
	private String host;
	private String userName;
	private String password;

	private Session sess;
	private InputStream in;
	private OutputStream out;
	private Connection conn;
	private RemoteConsumer consumer;
	private final StringBuilder shellText = new StringBuilder(1024 * 10);
	private StringBuilder shellLine = new StringBuilder(1024);
	private final Object lock = new Object();
	private boolean isRun = false;
	private long lastUpdateTime = System.currentTimeMillis();
	private final long startTime = System.currentTimeMillis();
	private List<String> listLog = null;
	private Object logLock = null;
	private final Object timeLock = new Object();
	private boolean isFirstCmd = true;

	public void startShell(final List<String> listLog, final Object logLock) throws Exception {
		this.listLog = listLog;
		this.logLock = logLock;
		this.isFirstCmd = true;

		this.conn = new Connection(host);
		conn.connect(); // make sure the connection is opened
		boolean isAuthenticated = conn.authenticateWithPassword(userName, password);
		if (isAuthenticated == false) {
			throw new IOException("Authentication failed.");
		}

		this.sess = conn.openSession();
		sess.requestPTY("dumb", 1000, 100, 0, 0, null);
		this.sess.startShell();

		this.in = sess.getStdout();
		this.out = sess.getStdin();
		this.isRun = true;
		this.consumer = new RemoteConsumer();
		this.consumer.start();
	}

	public void waitForEnd(final String user, final String cmdEndSymbol) throws Exception {
		char cc = 10;
		this.executeCmd(cc + "", user, cmdEndSymbol);
	}

	public void waitForEnd(final String user, final String cmdEndSymbol, final long maxWaitSeconds) throws Exception {
		char cc = 10;
		this.executeCmd(cc + "", user, cmdEndSymbol, maxWaitSeconds);
	}

	public void executeCmd(final String cmd, final String user, final String cmdEndSymbol) throws Exception {
		this.executeCmd(cmd, user, cmdEndSymbol, -1);
	}

	public void executeCmd(final String cmd, final String user, final String cmdEndSymbol, final long maxWaitSeconds) throws Exception {
		if (this.isFirstCmd == false) {
			boolean wait = true;
			long startWaitTime = System.currentTimeMillis();
			while (wait) {
				Thread.sleep(500);
				synchronized (lock) {
					String str = this.shellText.toString();
					if (str.trim().indexOf(cmdEndSymbol) >= 0) {
						int n1 = str.lastIndexOf("\n");
						// int n2 = str.lastIndexOf(user + "@");//
						// int n3 = str.lastIndexOf(user + "$");// ipa

						if (n1 > 0 && str.trim().endsWith("@")) {
							wait = false;
						}
						if (n1 > 0 && str.trim().endsWith("$")) {
							wait = false;
						}
					}
				}
				if (wait) {
					Thread.sleep(1000);
				}
				if (maxWaitSeconds > 0) {
					long nowTime = System.currentTimeMillis();
					if ((nowTime - startWaitTime) > maxWaitSeconds * 1000) {
						break;
					}
				}
			}
		} else {
			this.isFirstCmd = false;
		}

		// System.out.println("execute command:" + cmd);
		for (int i = 0; i < cmd.length(); i++) {
			char c = cmd.charAt(i);
			this.out.write(c);
		}
		out.write(10);
	}

	public void stopShell(long waitSeconds, long timeoutMinutes, final String endMatchStr) {
		if (waitSeconds <= 0) {
			waitSeconds = timeoutMinutes * 60;
		}
		if (timeoutMinutes <= 0) {
			timeoutMinutes = 30;
		}
		log.info("waitSeconds:" + waitSeconds + " timeoutMinutes=" + timeoutMinutes);
		boolean isWaiting = true;
		while (isWaiting) {
			try {
				Thread.sleep(1000);
				log.debug("waiting for end.....................");
				long nowTime = System.currentTimeMillis();
				synchronized (timeLock) {
//					isWaiting = (nowTime - this.lastUpdateTime) <= 1000 * waitSeconds && (nowTime - this.startTime) <= timeoutMinutes * 60000;
					log.debug("nowTime=" + nowTime + " this.lastUpdateTime=" + this.lastUpdateTime + "\r\n isWaiting=" + isWaiting + "  (nowTime-lastUpdateTime)=" + (nowTime - this.lastUpdateTime)
							+ "   (nowTime-startTime)=" + (nowTime - this.startTime));
					if (CM.stringIsNotEmpty(endMatchStr)) {
						if ((nowTime - this.lastUpdateTime) > 3000) {
							if (CtmpUtil.match(this.shellText.toString(), endMatchStr)) {
								isWaiting = false;
							}
						}
					}
					
					if ((nowTime - this.startTime) > timeoutMinutes * 60 * 1000){
						this.shellText.append("It's too long to deploy the app!!!");
						isWaiting = false;
					}
					
				}
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		log.info("send stop request............");
		try {
			out.write(3);
			Thread.sleep(500);
			this.isRun = false;
			out.write(10);
			Thread.sleep(300);
			out.write(10);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	public void stopRun() {
		this.isRun = false;
	}

	class RemoteConsumer extends Thread {
		private void addText(final byte[] data, final int len) {
			String str = new String(data, 0, len);
			synchronized (lock) {
				// System.out.println(str);
				shellText.append(str);
			}

			shellLine.append(str);
			int n = shellLine.toString().indexOf("\n");
			if (n > 0) {
				String lnStr[] = shellLine.toString().split("\n");
				for (int i = 0; i < lnStr.length - 1; i++) {
					if (lnStr[i].startsWith("\r") && lnStr[i].length() > 1) {
						lnStr[i] = lnStr[i].substring(1);
					}
					if (lnStr[i].endsWith("\r") && lnStr[i].length() > 1) {
						lnStr[i] = lnStr[i].substring(0, lnStr[i].length() - 1);
					}
					if (lnStr[i].length() > 0) {
						synchronized (logLock) {
							listLog.add(lnStr[i]);
						}
					}
				}
				shellLine = new StringBuilder(1024);
				shellLine.append(lnStr[lnStr.length - 1]);
			}

		}

		@Override
		public void run() {
			byte[] buff = new byte[1024 * 64];

			try {
				while (isRun) {
					int len = in.read(buff);
					if (len == -1) {
						log.error("consumer stopped!!!!!!!!!!!!!!!!!!");
						return;
					}
					synchronized (timeLock) {
//						log.debug("update the lastUpdateTime");
						lastUpdateTime = System.currentTimeMillis();
					}
					addText(buff, len);
				}
				synchronized (logLock) {
					if (CM.stringIsNotEmpty(shellLine.toString())) {
						listLog.add(shellLine.toString());
					}
				}
				System.out.println("Done!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
			} catch (Exception e) {
				log.error(e.getMessage(), e);
			}
		}
	}

	public String getHost() {
		return host;
	}

	public void setHost(final String host) {
		this.host = host;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(final String userName) {
		this.userName = userName;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(final String password) {
		this.password = password;
	}

	public StringBuilder getShellText() {
		return shellText;
	}

	/**
	 * @param args
	 */
	public static void main(final String[] args) {
		// TODO Auto-generated method stub
		URL configURL = SshService.class.getResource("/properties/log4j.properties");
		PropertyConfigurator.configure(configURL);

		boolean tmp1 = true;
		
		SshService ssh = new SshService();
		ssh.userName = "oracle";
		ssh.password = "st2@oracle";
		ssh.host = "192.168.63.219";
		
		try {			
			List<String> lt = new Vector<String>();
			Object lk = new Object();
//			
//			ssh.conn = new Connection(ssh.host);
//			ssh.conn.connect(); // make sure the connection is opened
			ssh.startShell(lt, lk);
//			while(tmp1){
//				Thread.sleep(1000);
//				
//			}
			String cmd = "/opt/tibco/ems/6.0/bin/tibemsadmin -script /opt/script/Tibco/CRL-15Q1-RYL222333_617_1423039361948.conf -server tcp://192.168.15.74:7222 -user admin -password admin";
			ssh.executeCmd(cmd, "oracle", "$");
			Thread.sleep(5000);
			ssh.stopRun();
//			System.out.println("Done!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
//			System.out.println(ssh.getShellText());
			
//			System.out.println(ssh.conn.authenticateWithPassword(ssh.userName, ssh.password));
			
			// ssh.startShell(lt, lk, null);
			// ssh.executeCmd("ll");
			// ssh.executeCmd("cd /opt/oracle/tomcat/cums-3/bin");
			// ssh.executeCmd("ll");
			// ssh.executeCmd("./startup.sh ");
			// ssh.executeCmd("tail -f ../logs/catalina.out ");
			// ssh.stopShell(2, 0);
//			System.out.println(ssh.shellText);
//			System.out.println("line------------------------------------------------");
//			for (String str : lt) {
//				System.out.println(str);
//			}
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
}