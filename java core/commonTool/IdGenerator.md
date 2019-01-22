主键生成器，目前这个再高并发下还存在重复的可能哦。还需要优化下。

~~~java
import java.net.InetAddress;
import java.text.NumberFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Random;
import java.util.UUID;
import java.util.concurrent.atomic.AtomicInteger;

public class IdGenerator {
    private static final String IP_AFTER_TWO;
    private static final AtomicInteger SEQ_ATOMIC_INTEGER = new AtomicInteger(0);
    private static final DateTimeFormatter DATE_FORMAT = DateTimeFormatter.ofPattern("yyMMddHHmmssSSS");

    private static final String EMPTY = "";
    private static final String ZERO = "0";
    private static final String DOUBLE_ZERO = "00";

    static {
        String ip;
        try {
            InetAddress inetAddress = InetAddress.getLocalHost();
            ip = inetAddress.getHostAddress();
            ip = ip.substring(ip.lastIndexOf(46) + 1);
            ip = formatNumber(Long.valueOf(ip), 2);
        } catch (Exception e) {
            Random random = new Random(UUID.randomUUID().toString().hashCode());
            int randomNum = random.nextInt(99);
            ip = formatNumber(randomNum, 2);
        }
        IP_AFTER_TWO = ip;
    }

    private static String formatNumber(long number, int digits) {
        NumberFormat nf = NumberFormat.getInstance();
        nf.setMaximumIntegerDigits(digits);
        nf.setMinimumIntegerDigits(digits);
        nf.setGroupingUsed(false);
        return nf.format(number);
    }

    private static int getSeq() {
        int result = SEQ_ATOMIC_INTEGER.incrementAndGet();
        if (result <= 999) {
            return result;
        }
        SEQ_ATOMIC_INTEGER.set(1);
        return 1;
    }

    private static String getSequence() {
        int seq = getSeq();
        return (seq < 10 ? DOUBLE_ZERO : (seq < 100 ? ZERO : EMPTY)) + seq;
    }

    public static String generate() {
        return generate("", "");
    }

    /**
     * @param prefix 前缀
     */
    public static String generate(String prefix) {
        return generate(prefix, "");
    }

    /**
     * @param prefix 前缀
     * @param suffix 后缀
     */
    public static String generate(String prefix, String suffix) {
        StringBuilder buffer = new StringBuilder();
        String date = LocalDateTime.now().format(DATE_FORMAT);
        buffer.append(prefix)
                .append(date)
                .append(IP_AFTER_TWO)
                .append(getSequence())
                .append(suffix);
        return buffer.toString();
    }
}
~~~

