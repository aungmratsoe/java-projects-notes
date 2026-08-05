# Server IP ကို Hardcode မထားဘဲ Dynamic ဖြစ်အောင် လုပ်နည်း

Server IP ကို client code ထဲမှာ hardcode မလုပ်ဘဲ ရအောင် နည်းလမ်း **၄ မျိုး** ရှိပါတယ် — Project ရဲ့ လိုအပ်ချက်အလိုက် ရွေးလို့ရပါတယ်.

## Option 1 — Config File (`.properties`) — အလွယ်ဆုံး၊ အသုံးအများဆုံး

Client jar နဲ့ အတူ config file ကို ချန်ထားပြီး, Server IP ပြောင်းချင်ရင် **code ပြန် compile မလိုဘဲ** file ကိုပဲ ပြင်လို့ရပါတယ်.

```properties
# config.properties (Client jar ရှိတဲ့ folder ထဲမှာ ထားရမယ်)
server.ip=192.168.1.10
server.port=1099
```

```java
// com.ams.qrcode.client.ConfigLoader.java
package com.ams.qrcode.client;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class ConfigLoader {
    private static Properties props = new Properties();

    static {
        try (FileInputStream fis = new FileInputStream("config.properties")) {
            props.load(fis);
        } catch (IOException e) {
            // File မရှိရင် default value သုံး
            props.setProperty("server.ip", "localhost");
            props.setProperty("server.port", "1099");
        }
    }

    public static String getServerIp() {
        return props.getProperty("server.ip");
    }

    public static int getServerPort() {
        return Integer.parseInt(props.getProperty("server.port", "1099"));
    }
}
```

```java
// RmiConnectionManager.java ထဲမှာ
public void connect() throws Exception {
    String ip = ConfigLoader.getServerIp();
    int port = ConfigLoader.getServerPort();
    Registry registry = LocateRegistry.getRegistry(ip, port);
    userService = (UserService) registry.lookup("UserService");
    studentService = (StudentService) registry.lookup("StudentService");
}
```

**Client computer တစ်လုံးချင်းစီမှာ** `config.properties` ရဲ့ `server.ip` ကို ပြင်ပေးရုံနဲ့ Server ပြောင်းလဲသွားလည်း Client app ကို ပြန် compile မလိုပါ။

## Option 2 — Connect Dialog (UI ထဲက Login/Connect screen)

SignIn form မတိုင်ခင် **"Server Settings"** dialog တစ်ခု ပြပြီး User ကိုယ်တိုင် IP ထည့်ခိုင်းတာ — Part 2 က RMI Client GUI ဥပမာမှာ ပြပြီးသားပါ:

```java
// App စဖွင့်ချင်းမှာ ပြမယ့် dialog
public class ServerConnectDialog extends JDialog {
    private JTextField ipField = new JTextField("192.168.1.10", 15);
    private JButton connectButton = new JButton("Connect");

    // Connect button click ရင် RmiConnectionManager.connect(ipField.getText()) ခေါ်
}
```

**အားသာချက်**: User (office staff) ကို IP field ပြင်ခွင့်ပေးလို့ Server ပြောင်းရင် app ပြန် install/config file ပြင်စရာမလိုပါ။ **အားနည်းချက်**: User က IP နံပါတ် သိထားရမယ် (Technical knowledge နည်းနည်းလို)။

## Option 3 — Auto Network Discovery (UDP Broadcast) — အကောင်းဆုံး UX

Server ကို **manual IP ရိုက်စရာမလိုအောင်**, Client က WiFi network ပေါ်က Server ကို **အလိုအလျောက် ရှာတွေ့** အောင် UDP broadcast သုံးလို့ရပါတယ်:

```java
// com.ams.qrcode.server.ServerDiscoveryListener.java (Server ဘက်)
import java.net.*;

public class ServerDiscoveryListener implements Runnable {
    private static final int DISCOVERY_PORT = 8888;

    @Override
    public void run() {
        try (DatagramSocket socket = new DatagramSocket(DISCOVERY_PORT)) {
            byte[] buf = new byte[256];
            while (true) {
                DatagramPacket packet = new DatagramPacket(buf, buf.length);
                socket.receive(packet); // Client ကနေ "DISCOVER_QRSERVER" ရောက်လာအောင် စောင့်

                String received = new String(packet.getData(), 0, packet.getLength());
                if (received.equals("DISCOVER_QRSERVER")) {
                    // Reply — server ရဲ့ actual IP ကို ပြန်ပို့
                    String myIp = InetAddress.getLocalHost().getHostAddress();
                    byte[] response = myIp.getBytes();
                    DatagramPacket responsePacket = new DatagramPacket(
                        response, response.length, packet.getAddress(), packet.getPort());
                    socket.send(responsePacket);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
// ServerMain.java ထဲမှာ background thread အနေနဲ့ စတင်
new Thread(new ServerDiscoveryListener()).start();
```

```java
// com.ams.qrcode.client.ServerDiscovery.java (Client ဘက်)
import java.net.*;

public class ServerDiscovery {
    public static String findServerIp() throws Exception {
        try (DatagramSocket socket = new DatagramSocket()) {
            socket.setBroadcast(true);
            socket.setSoTimeout(3000); // 3 seconds အတွင်း reply မရရင် timeout

            byte[] sendData = "DISCOVER_QRSERVER".getBytes();
            DatagramPacket sendPacket = new DatagramPacket(
                sendData, sendData.length, 
                InetAddress.getByName("255.255.255.255"), 8888); // WiFi network တစ်ခုလုံးဆီ broadcast
            socket.send(sendPacket);

            byte[] recvBuf = new byte[256];
            DatagramPacket recvPacket = new DatagramPacket(recvBuf, recvBuf.length);
            socket.receive(recvPacket); // Server ရဲ့ reply ကို စောင့်

            return new String(recvPacket.getData(), 0, recvPacket.getLength());
        }
    }
}
```

```java
// RmiConnectionManager.java ထဲမှာ
public void connect() throws Exception {
    String ip = ServerDiscovery.findServerIp(); // Auto-discover
    Registry registry = LocateRegistry.getRegistry(ip, 1099);
    // ...
}
```

**အားသာချက်**: User က IP ဘာမှ သိစရာမလို — Client app ကို run လိုက်ရင် Server ကို auto-connect ဖြစ်သွားမယ်။ **Trade-off**: Code ပိုရှုပ်တယ်, Firewall မှာ UDP port 8888 ထပ်ဖွင့်ရမယ်, Router ရဲ့ broadcast settings အပေါ် မူတည်ပါတယ် (WiFi client isolation ဖွင့်ထားရင် ဒါလည်း အလုပ်မလုပ်ပါ).

## Option 4 — Hostname (DNS name) — Static IP အစား Name သုံးခြင်း

Server computer မှာ **static hostname** သတ်မှတ်ထားပြီး (router ရဲ့ local DNS, `/etc/hosts` mDNS/Bonjour) Client ဘက်က IP အစား name သုံးနိုင်ပါတယ်:

```java
// Windows/Mac zeroconf (mDNS) ရှိရင်
Registry registry = LocateRegistry.getRegistry("qrserver.local", 1099);
```

Router setting (DHCP reservation) ထဲမှာ Server computer ကို hostname `qrserver` လို့ assign ထားရင် IP ပြောင်းသွားလည်း Client code ပြောင်းစရာမလိုပါ။ (Router setup လိုအပ်တာကြောင့် office/school environment အတွက် ပိုသင့်တော်ပါတယ်)

## ဘယ်ဟာ ရွေးသင့်လဲ

|Situation|Recommended|
|---|---|
|Setup လွယ်၊ Server IP မကြာခဏ မပြောင်း|Option 1 (Config file)|
|Non-technical user တွေ Server ချိတ်ဖို့|Option 3 (Auto discovery)|
|Technical support/Admin user|Option 2 (Connect dialog)|
|Server IP fix ဖြစ်နေတဲ့ office network|Option 4 (Hostname/DHCP reservation)|

**Practical suggestion**: **Option 1 + Option 2 ပေါင်းသုံး** — Config file ကနေ default IP ဖတ်၊ Connect fail ဖြစ်ရင် dialog ပြပြီး User ကို IP ပြန်ရိုက်ခိုင်းတာက balance ကောင်းပါတယ် (Auto-load + Manual override).

---

Config file + Connect dialog ပေါင်းထားတဲ့ full implementation (fallback logic ပါ) ကူညီရေးပေးရမလား၊ ဒါမှမဟုတ် Auto-discovery (Option 3) ကို firewall setup အသေးစိတ်ထိ ဆက်ရှင်းပြရမလား?