public static void main(String[] argv) {
   

        /* handle kill or ctrl-c command from os */
        SignalHandler handler = new SignalHandler() {
            public void handle(Signal signal) {
                log.info("get terminal signal from os");
            
                log.info("bye!");
                System.exit(0);
            }
        };

        Signal.handle(new Signal("TERM"), handler);// 相当于kill
        Signal.handle(new Signal("INT"), handler);// 相当于Ctrl+C
    }