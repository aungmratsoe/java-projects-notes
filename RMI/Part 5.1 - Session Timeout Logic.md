# Session Timeout Logic — အသေးစိတ် — Part 5.1

Part 5 က `activeSessions` Map ထဲမှာ token တွေကို **အချိန်မရွေး valid** အဖြစ် သိမ်းထားခဲ့ပါတယ် — ဒါက security risk ပါ (Client တစ်ယောက် login ဝင်ပြီး app ပိတ်မထားခဲ့ရင် token က ထာဝရ အသုံးပြုလို့ရနေမယ်)။ Session timeout ထည့်ဖို့ လိုအပ်တဲ့ concept တွေကို အသေးစိတ် လုပ်ပြပါမယ်။

## Timeout Strategy ၂ မျိုး

|Strategy|ရှင်းလင်းချက်|
|---|---|
|**Absolute timeout**|Login ဝင်ပြီးတဲ့ အချိန်ကနေ ဂဏန်းအတိအကျ (ဥပမာ 8 နာရီ) ကြာရင် expire — client က active ဖြစ်နေသေးလည်း logout ဖြစ်သွားမယ်|
|**Sliding (idle) timeout**|Method ခေါ်တိုင်း "last access time" ကို update လုပ်၊ **idle** အချိန် (ဥပမာ 15 မိနစ်) ကျော်မှ expire — ပိုအသုံးများပါတယ်|

လက်တွေ့မှာ **နှစ်ခုစလုံး ပေါင်းသုံး**တာ အကောင်းဆုံးပါ (ဥပမာ - idle 15 မိနစ်, absolute max 8 နာရီ)။

## Step 1 — Session Object ဖန်တီးမယ် (token string ချည်းမကျန်ဘဲ metadata ပါ)

```java
// SessionInfo.java
public class SessionInfo {
    private final String username;
    private final long loginTime;      // Absolute timeout တွက်ဖို့
    private volatile long lastAccessTime; // Sliding timeout တွက်ဖို့

    public SessionInfo(String username) {
        this.username = username;
        this.loginTime = System.currentTimeMillis();
        this.lastAccessTime = this.loginTime;
    }

    public void touch() {
        this.lastAccessTime = System.currentTimeMillis();
    }

    public boolean isExpired(long idleTimeoutMs, long absoluteTimeoutMs) {
        long now = System.currentTimeMillis();
        boolean idleExpired = (now - lastAccessTime) > idleTimeoutMs;
        boolean absoluteExpired = (now - loginTime) > absoluteTimeoutMs;
        return idleExpired || absoluteExpired;
    }

    public String getUsername() { return username; }
}
```

## Step 2 — AuthImpl ကို Timeout Logic နဲ့ ပြင်မယ်

```java
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class AuthImpl extends UnicastRemoteObject implements AuthInterface {

    private final Map<String, SessionInfo> activeSessions = new ConcurrentHashMap<>();

    // Timeout ကာလများ (millisecond)
    private static final long IDLE_TIMEOUT = 15 * 60 * 1000;       // 15 မိနစ်
    private static final long ABSOLUTE_TIMEOUT = 8 * 60 * 60 * 1000; // 8 နာရီ

    private final Map<String, String> validUsers = Map.of(
        "admin", "admin123",
        "branch1", "branch1pass"
    );

    protected AuthImpl() throws RemoteException {
        super();
        startCleanupTask(); // Background ကနေ expired session တွေ အလိုအလျောက် ရှင်းမယ်
    }

    @Override
    public String login(String username, String password) throws RemoteException {
        String correctPassword = validUsers.get(username);
        if (correctPassword == null || !correctPassword.equals(password)) {
            throw new SecurityException("Username (သို့) Password မှားနေပါသည်");
        }
        String token = UUID.randomUUID().toString();
        activeSessions.put(token, new SessionInfo(username));
        return token;
    }

    // Method ခေါ်တိုင်း ဒီကို ခေါ်ပြီး session validity + timeout စစ်ရမယ်
    public boolean isValidSession(String token) {
        SessionInfo session = activeSessions.get(token);
        if (session == null) return false;

        if (session.isExpired(IDLE_TIMEOUT, ABSOLUTE_TIMEOUT)) {
            activeSessions.remove(token); // Expire ဖြစ်ရင် ချက်ချင်း ဖယ်ရှား
            return false;
        }

        session.touch(); // ✅ Valid access ဆိုရင် "last access time" ကို update (sliding timeout)
        return true;
    }

    @Override
    public void logout(String token) throws RemoteException {
        activeSessions.remove(token);
    }

    // === Background Cleanup Task ===
    // Idle ဖြစ်နေတဲ့ session တွေကို client က method ထပ်မခေါ်တော့ရင်တောင် 
    // memory ထဲက ရှင်းချင်လို့ scheduled task နဲ့ periodically စစ်ဖျက်မယ်
    private void startCleanupTask() {
        ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
        scheduler.scheduleAtFixedRate(() -> {
            int before = activeSessions.size();
            activeSessions.entrySet().removeIf(
                entry -> entry.getValue().isExpired(IDLE_TIMEOUT, ABSOLUTE_TIMEOUT)
            );
            int removed = before - activeSessions.size();
            if (removed > 0) {
                System.out.println("Expired session " + removed + " ခု ရှင်းလိုက်ပါသည်");
            }
        }, 1, 1, TimeUnit.MINUTES); // 1 မိနစ်ခြားပြီး check
    }
}
```

## Step 3 — Interface ကို `logout()` ထပ်ထည့်

```java
public interface AuthInterface extends Remote {
    String login(String username, String password) throws RemoteException;
    void logout(String token) throws RemoteException;
}
```

## Step 4 — Client ဘက်က Session Expired ဖြစ်ရင် ကိုင်တွယ်ပုံ

Server က `SecurityException("Login ဝင်ထားခြင်း မရှိပါ...")` throw လိုက်တဲ့အခါ Client Swing GUI ဘက်က ဒီလို catch လုပ်ပြီး login screen ကို ပြန်ပို့ရပါမယ်:

```java
// InventoryClient method ခေါ်တဲ့နေရာ (SwingWorker done() ထဲ)
protected void done() {
    try {
        boolean result = get();
        resultLabel.setText("Success: " + result);
    } catch (Exception ex) {
        Throwable cause = ex.getCause();
        if (cause instanceof SecurityException) {
            // Session expire ဖြစ်နေပြီ → Login dialog ပြန်ပြ
            JOptionPane.showMessageDialog(RmiClientGUI.this,
                "Session သက်တမ်းကုန်သွားပါပြီ၊ ပြန်ဝင်ပါ", 
                "Session Expired", JOptionPane.WARNING_MESSAGE);
            showLoginDialog(); // login window ပြန်ဖွင့်
        } else {
            resultLabel.setText("Error: " + cause);
        }
    }
}
```

## Client ဘက်က Auto-Logout Warning ပြသနည်း (Optional Enhancement)

Client GUI မှာ **Swing Timer** သုံးပြီး idle timeout မတိုင်ခင် warning ပြနိုင်ပါတယ်:

```java
import javax.swing.Timer;

// 13 မိနစ်နေရင် (idle timeout 15 မိနစ်ရဲ့ 2 မိနစ်ကြိုတင်) warning ပြမယ်
Timer idleWarningTimer = new Timer(13 * 60 * 1000, e -> {
    int choice = JOptionPane.showConfirmDialog(this,
        "Session 2 မိနစ်အတွင်း ကုန်တော့မှာပါ၊ ဆက်လက်အသုံးပြုမလား?",
        "Session Warning", JOptionPane.YES_NO_OPTION);
    if (choice == JOptionPane.YES_OPTION) {
        // Dummy call တစ်ခု ပို့ပြီး session ကို "touch" လုပ်ခိုင်း (sliding timeout refresh)
        refreshSession();
    }
});
idleWarningTimer.setRepeats(false);

// User action (button click) တိုင်းမှာ timer ကို reset
private void onUserActivity() {
    idleWarningTimer.restart();
}
```

## အနှစ်ချုပ် — Timeout Logic Checklist

|အချက်|အကြောင်းအရာ|
|---|---|
|`lastAccessTime` ကို method ခေါ်တိုင်း update|Sliding timeout အတွက်|
|`loginTime` ကို absolute cap အတွက် ခွဲသိမ်း|Idle မဖြစ်သေးလည်း max session length ထား|
|`ConcurrentHashMap` သုံး|Client များစွာ တစ်ပြိုင်နက် login/access လုပ်နိုင်လို့|
|`ScheduledExecutorService` နဲ့ background cleanup|Memory ထဲမှာ dead session တွေ စုမနေအောင်|
|`isValidSession()` ကို method **တိုင်း** ရဲ့ အစမှာ ခေါ်|Skip လုပ်လို့ မရအောင် consistently စစ်ရမယ်|
|Client ဘက်က `SecurityException` catch ပြီး login ပြန်ပို့|UX ကောင်းအောင်|

---

RMI security အပိုင်း ပြည့်စုံသွားပါပြီ။ နောက်တစ်ခု ဆက်ချင်တာ ရှိပါသလား — **Part 1 က ရပ်ထားခဲ့တဲ့ gRPC learning** ကို ပြန်ဆက်မလား၊ ဒါမှမဟုတ် တခြား Java topic တစ်ခုခု စချင်ပါသလား?