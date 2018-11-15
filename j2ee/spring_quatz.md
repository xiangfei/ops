<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

    <bean id="util" class="com.newcosoft.lsmp.report.schedule.Util" autowire="byName" />

    <bean name="dayJob" class="org.springframework.scheduling.quartz.JobDetailBean">
        <property name="jobClass" value="com.newcosoft.lsmp.report.schedule.JobProcessor" />
        <property name="jobDataAsMap">
            <map>
                <entry key="util" value-ref="util" />
                <entry key="procedureList0">
                    <list>
                        <value>SP_STAT_BAWARD_DETAIL</value>
                        <value>SP_STAT_AWARD_ADJUST_FUND</value>
                        <value>SP_STAT_TAX_AREA_DAY</value>
                        <value>SP_STAT_PERCONSUME_BET_DAY</value>
                        <value>SP_STAT_CONSUME_TOTAL_DAY</value>
                        <value>SP_STAT_USER_BETCHAN_DAY</value>
                        <value>SP_STAT_AREA_SALE_DAY</value><!-- 20111212 yesterday string-->
                        <value>SP_STAT_USER_DAY</value>
                        <value>SP_STAT_USER_ABNORMAL</value>
                        <value>SP_STAT_DRAW_REQ</value>
                    </list>
                </entry>
                <entry key="procedureList1">
                    <list>
                        <value>SP_STAT_CASH_ACCOUNT</value><!-- int out-->
                        <value>SP_STAT_CHARGE_ACCOUNT</value>
                    </list>
                </entry>
                <entry key="procedureList2">
                    <list>
                        <value>SP_STAT_TIMESPAN</value><!-- 20111212 yesterday string,int out-->
                        <value>SP_STAT_RUN_ISSUE_PROCEDURE</value>
                    </list>
                </entry>
            </map>
        </property>
    </bean>
    <bean name="weekJob" class="org.springframework.scheduling.quartz.JobDetailBean">
        <property name="jobClass" value="com.newcosoft.lsmp.report.schedule.JobProcessor" />
        <property name="jobDataAsMap">
            <map>
                <entry key="util" value-ref="util" />
                <entry key="procedureList0">
                    <list>
                        <value>SP_STAT_AREA_SALE_WEEK</value><!-- 20111212 yesterday string-->
                    </list>
                </entry>
            </map>
        </property>
    </bean>
    <bean name="monthJob" class="org.springframework.scheduling.quartz.JobDetailBean">
        <property name="jobClass" value="com.newcosoft.lsmp.report.schedule.JobProcessor" />
        <property name="jobDataAsMap">
            <map>
                <entry key="util" value-ref="util" />
                <entry key="procedureList1">
                    <list>
                        <value>SP_STAT_USER_LEVEL</value><!-- int out-->
                    </list>
                </entry>
                <entry key="procedureList3">
                    <list>
                        <value>SP_STAT_AREA_SALE_MONTH</value><!-- 201111 yesterday string-->
                        <value>SP_STAT_TAX_AREA_MONTH</value>
                        <value>SP_STAT_PERCONSUME_BET_MONTH</value>
                        <value>SP_STAT_CONSUME_TOTAL_MONTH</value>
                        <value>SP_STAT_USER_BETCHAN_MONTH</value>
                        <value>SP_STAT_USER_MONTH</value>
                    </list>
                </entry>
            </map>
        </property>
    </bean>

    <!-- Seconds Minutes Hours Day-of-month Month Day-of-Week (Year)-->
    <bean id="dayTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
        <property name="jobDetail" ref="dayJob" />
        <property name="cronExpression" value="0 0 1 * * ?" /><!-- 0 0 1 * * ? -->
    </bean>
    <bean id="weekTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
        <property name="jobDetail" ref="weekJob" />
        <property name="cronExpression" value="0 2 1 ? * MON" /><!-- 0 2 1 ? * MON -->
    </bean>
    <bean id="monthTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
        <property name="jobDetail" ref="monthJob" />
        <property name="cronExpression" value="0 4 1 1 * ?" /><!-- 0 4 1 1 * ? -->
    </bean>
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="dayTrigger" />
                <ref bean="weekTrigger" />
                <ref bean="monthTrigger" />
            </list>
        </property>
    </bean>

</beans>

package com.newcosoft.lsmp.report.schedule;

import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
import java.sql.Types;
import java.util.List;

import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;

/**
 * @className JobProcessor
 * @description
 * @author Tom
 * @date 2011-12-9 下午4:48:14
 * 
 */
public class JobProcessor extends QuartzJobBean {
    private static final Logger log = LoggerFactory.getLogger(JobProcessor.class);
    private Util util;
    private Connection conn;
    private CallableStatement cstmt;

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        JobDataMap map = context.getMergedJobDataMap();
        jdbcGetConn();
        for (int i = 0; i < 4; i++) {
            List<String> pList = (List<String>) map.get("procedureList" + i);
            if (pList != null) {
                for (String p : pList) {
                    runProcedure(p, i);
                }
            }
        }
        jdbcClose(conn, cstmt);
    }

    private void runProcedure(String procedure, int type) {
        log.info(type + " running " + procedure);
        try {
            if (type == 0) {
                cstmt = conn.prepareCall("{call " + procedure + "(?)}");
                cstmt.setString(1, Util.getStringDate("yyyyMMdd"));
            } else if (type == 1) {
                cstmt = conn.prepareCall("{call " + procedure + "(?)}");
                cstmt.registerOutParameter(1, Types.NUMERIC);
            } else if (type == 2) {
                cstmt = conn.prepareCall("{call " + procedure + "(?,?)}");
                cstmt.setString(1, Util.getStringDate("yyyyMMdd"));
                cstmt.registerOutParameter(2, Types.NUMERIC);
            } else if (type == 3) {
                cstmt = conn.prepareCall("{call " + procedure + "(?)}");
                cstmt.setString(1, Util.getStringDate("yyyyMM"));
            }
            cstmt.execute();
        } catch (Exception e) {
            log.error(procedure + " Exception:", e);
        }
    }

    public void jdbcGetConn() {
        try {
            Class.forName(util.getDriverClassName());
            conn = DriverManager.getConnection(util.getUrl(), util.getUsername(), util.getPassword());
        } catch (Exception e) {
            log.error("Exception:", e);
        }
    }

    public void jdbcClose(Connection conn, Statement stmt) {
        try {
            if (conn != null)
                conn.close();
            if (stmt != null)
                stmt.close();
        } catch (Exception e) {
            log.error("Exception:", e);
        }
    }

    public void setUtil(Util util) {
        this.util = util;
    }

}





package com.newcosoft.lsmp.report.schedule;

import java.io.IOException;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Properties;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.newcosoft.lsmp.common.setting.LsmpSetting;

/**
 * @className Util
 * @description
 * @author Tom
 * @date 2011-12-10 下午1:36:21
 * 
 */
public class Util {
    private static final Logger log = LoggerFactory.getLogger(Util.class);
    private String driverClassName = "oracle.jdbc.driver.OracleDriver";
    private String url;
    private String username;
    private String password;

    public Util() {
        Properties systemParameters = new Properties();
        try {
            systemParameters.load(LsmpSetting.class.getResourceAsStream("/report-web.properties"));
            url = systemParameters.getProperty("db_url");
            username = systemParameters.getProperty("db_user");
            password = systemParameters.getProperty("db_password");
        } catch (IOException e) {
            log.error("IOException:", e);
        }
    }

    public static String getStringDate(String pattern) {
        Calendar cal = Calendar.getInstance();
        cal.add(Calendar.DAY_OF_MONTH, -1);
        DateFormat format = new SimpleDateFormat(pattern);
        return format.format(cal.getTime());
    }

    public void setDriverClassName(String driverClassName) {
        this.driverClassName = driverClassName;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getDriverClassName() {
        return driverClassName;
    }

    public String getUrl() {
        return url;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }

}



public class LsmpScheduleMain {

    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("beans-scheduler.xml");
    }
}