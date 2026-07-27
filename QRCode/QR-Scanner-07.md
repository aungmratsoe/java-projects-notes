
အခုရောက်လာတဲ့ **Chapter 7 — Webcam Loop** က ဒီ Project ရဲ့ **Heart of the Scanner** ဖြစ်ပါတယ်။

ဒီ Loop က Camera ကိုဖွင့်ပြီး

- Live Video ရယူတယ်
    
- GUI မှာ ပြတယ်
    
- QR Code ကို Scan လုပ်တယ်
    
- Result ရရင် Verify လုပ်တယ်
    

အားလုံးကို ထိန်းချုပ်ပါတယ်။

ဒီ Chapter ကို နားလည်သွားရင်

- Webcam Programming
    
- Infinite Loop
    
- Live Video Streaming
    
- Frame Processing
    
- Background Thread
    
- QR Detection Pipeline
    

အားလုံးကို နားလည်သွားမယ်။

---

# Chapter Overview

ဒီ Code ကို လေ့လာမယ်။

```java
executor.execute(() -> {

    try {

        webcam = Webcam.getDefault();

        if (webcam == null) {
            ...
            return;
        }

        webcam.setViewSize(WebcamResolution.VGA.getSize());

        webcam.open();

        while (isRunning) {

            if (!webcam.isOpen()) {
                break;
            }

            BufferedImage image = webcam.getImage();

            if (image != null) {

                Image scaledImg = image.getScaledInstance(...);

                SwingUtilities.invokeLater(...);

                if (!isDialogShowing.get()) {

                    Result result = decodeQRCode(image);

                    if (result != null) {

                        ...

                    }

                }

            }

            Thread.sleep(40);

        }

    } catch (...) {

    }

});
```

ဒီ Code တစ်ခုတည်းက Scanner System တစ်ခုလုံးကို မောင်းနှင်နေပါတယ်။

---

# Overall Flow

```text
Create Thread

      │

      ▼

Find Webcam

      │

      ▼

Open Webcam

      │

      ▼

while(isRunning)

      │

      ▼

Capture Frame

      │

      ▼

Display Frame

      │

      ▼

Decode QR

      │

      ▼

QR Found?

      │

 ┌────┴────┐

 │         │

No        Yes

 │         │

 ▼         ▼

Next     Verify QR

Frame

 │

 ▼

Sleep

 │

 ▼

Repeat
```

---

# Step 1 — executor.execute()

```java
executor.execute(() -> {
```

Chapter 6 မှာ ရှင်းခဲ့သလို

ဒီ Code အားလုံးဟာ

**Background Thread** ပေါ်မှာ Run နေပါတယ်။

Diagram

```text
Main Thread

↓

GUI

--------------------

Worker Thread

↓

Webcam

↓

QR Decode
```

ဒါကြောင့် GUI မ Freeze ဘူး။

---

# Step 2 — Get Default Camera

```java
webcam = Webcam.getDefault();
```

ဒီ Line က

Computer မှာရှိတဲ့

Default Camera

ကို ရှာတယ်။

ဥပမာ

```text
Laptop

↓

Integrated Camera
```

ဒါမှမဟုတ်

```text
USB Webcam

↓

Logitech Camera
```

Default Camera ကို Return ပြန်ပေးတယ်။

---

Memory

အစမှာ

```text
webcam

↓

null
```

ပြီးတော့

```java
webcam = Webcam.getDefault();
```

လုပ်ပြီးရင်

```text
webcam

↓

Camera Object
```

ဖြစ်သွားတယ်။

---

# Step 3 — Camera Exists?

```java
if (webcam == null)
```

ဘာကြောင့် စစ်တာလဲ?

Camera မရှိနိုင်ဘူး။

ဥပမာ

- Desktop မှာ Webcam မတပ်ထားဘူး။
    
- Driver မရှိဘူး။
    
- Camera Disable လုပ်ထားတယ်။
    

---

Diagram

```text
Find Camera

      │

 ┌────┴────┐

 │         │

Found    Not Found

 │         │

 ▼         ▼

Open    Error Message
```

---

ဒီ Code

```java
SwingUtilities.invokeLater(() ->
        lblCam.setText("No Webcam Detected!"));
```

GUI မှာ

```text
No Webcam Detected!
```

ပြမယ်။

---

ပြီးတော့

```java
animationTimer.stop();
```

Animation ကိုလည်း ရပ်မယ်။

---

ပြီးတော့

```java
return;
```

Thread ကို

အဆုံးသတ်လိုက်တယ်။

---

# Step 4 — Set Resolution

```java
webcam.setViewSize(
        WebcamResolution.VGA.getSize());
```

ဒါက

Camera Resolution

သတ်မှတ်တာ။

---

VGA

=

```text
640 × 480
```

---

Diagram

```text
Camera

↓

640 × 480

↓

Capture Image
```

---

Resolution ကြီးရင်

```text
1920×1080

↓

Quality ↑

Speed ↓
```

Resolution သေးရင်

```text
320×240

↓

Speed ↑

Quality ↓
```

---

VGA က

Scanner အတွက် Balance ကောင်းတယ်။

---

# Step 5 — Open Camera

```java
webcam.open();
```

ဒီ Line က

Camera ကို

အမှန်တကယ် ဖွင့်တယ်။

Diagram

```text
Camera

OFF

↓

open()

↓

ON
```

---

ဒီအချိန်မှာ

Laptop Webcam LED

လင်းလာတတ်တယ်။

---

# Step 6 — Main Loop

```java
while(isRunning)
```

ဒီ Loop က

Scanner ရဲ့ Main Engine ပါ။

---

Flow

```text
Capture

↓

Display

↓

Decode

↓

Sleep

↓

Capture

↓

Display

↓

Decode

↓

Repeat
```

ဒီ Loop က

Window ပိတ်တဲ့အထိ

Run နေတယ်။

---

# Step 7 — Camera Still Open?

```java
if (!webcam.isOpen())
```

ဘာကြောင့် စစ်တာလဲ?

Camera ကို

User

ဒါမှမဟုတ်

OS

က ပိတ်လိုက်နိုင်တယ်။

ဥပမာ

```text
Webcam

↓

Disconnected
```

---

ဒါဆို

```java
break;
```

Loop ကနေ

ထွက်သွားတယ်။

---

# break vs return

```java
break;
```

↓

Loop ပဲ ရပ်တယ်။

---

```java
return;
```

↓

Method တစ်ခုလုံး

ထွက်သွားတယ်။

---

Diagram

```text
while()

↓

break

↓

Continue after loop
```

```text
Method

↓

return

↓

Method End
```

---

# Step 8 — Capture Image

```java
BufferedImage image =
        webcam.getImage();
```

ဒီ Line က

Camera Frame

တစ်ခု ယူတာ။

---

Camera က

Video မပေးဘူး။

သူပေးတာ

Frame

တွေပဲ။

ဥပမာ

```text
Frame1

Frame2

Frame3

Frame4

Frame5
```

ဒီ Frame တွေကို

မြန်မြန်ဆက်တိုက် ပြရင်

Video လိုမြင်ရတာပါ။

---

Memory

```text
image

↓

BufferedImage
```

---

BufferedImage ထဲမှာ

Pixel

အားလုံးရှိတယ်။

---

# Step 9 — Image Exists?

```java
if(image != null)
```

Camera က

Frame မရနိုင်တဲ့အချိန်ရှိတယ်။

ဥပမာ

```text
Camera Starting...

↓

null
```

ဒါကြောင့်

Null Check

လုပ်ထားတာ။

---

# Step 10 — Resize

```java
Image scaledImg =
image.getScaledInstance(
...
Image.SCALE_SMOOTH);
```

Camera Frame က

640×480

ဖြစ်တယ်။

ဒါပေမယ့်

JLabel က

242×264

ဖြစ်နိုင်တယ်။

---

ဒါကြောင့်

Resize

လုပ်တယ်။

```text
640×480

↓

242×264
```

---

`Image.SCALE_SMOOTH`

ကို သုံးတာကြောင့်

Image Quality

ပိုကောင်းတယ်။

---

# Step 11 — Update GUI

```java
SwingUtilities.invokeLater(() -> {

    lblCam.setIcon(new ImageIcon(scaledImg));

});
```

Background Thread က

GUI ကို

တိုက်ရိုက် မပြင်ဘူး။

Diagram

```text
Worker Thread

↓

Image Ready

↓

invokeLater()

↓

EDT

↓

JLabel
```

ဒါက Swing Programming ရဲ့ Rule ပါ။

---

# Step 12 — Dialog Check

```java
if(!isDialogShowing.get())
```

Dialog ဖွင့်ထားရင်

QR Scan

မလုပ်ဘူး။

---

ဘာကြောင့်?

ဒီလိုမစစ်ရင်

```text
Popup

Popup

Popup

Popup
```

အများကြီး ပေါ်လာမယ်။

---

# Step 13 — Decode QR

```java
Result result =
decodeQRCode(image);
```

ဒီမှာ

Image ထဲက

QR ကို ရှာတယ်။

Flow

```text
BufferedImage

↓

ZXing

↓

Decode

↓

Result
```

---

QR မရှိရင်

```text
result

↓

null
```

---

QR ရှိရင်

```text
result

↓

QR Data
```

---

# Step 14 — QR Found?

```java
if(result != null)
```

ဒါက

QR တွေ့ပြီလား?

စစ်တာ။

---

တွေ့ရင်

```java
isDialogShowing.set(true);
```

Dialog Lock

လုပ်တယ်။

---

ပြီးတော့

```java
String qrData =
result.getText();
```

QR ထဲက

Text

ယူတယ်။

ဥပမာ

```text
StudentID:ST001
Token:ABC123
```

ဒါမှမဟုတ်

Encrypted Data

ဖြစ်နိုင်တယ်။

---

# Step 15 — Verify

```java
SwingUtilities.invokeLater(() -> {

    onQrCodeScanned(qrData);

});
```

ဒီ Method ထဲမှာ

- AES Decrypt
    
- Database Check
    
- Student Verify
    

အားလုံး လုပ်တယ်။

---

# Step 16 — Delay

```java
Thread.sleep(1500);
```

ဘာကြောင့်?

QR ကို

တွေ့ပြီးတာနဲ့

Camera က

နောက် Frame မှာ

ထပ်တွေ့နိုင်တယ်။

---

ဒါကြောင့်

1.5 Second

နားတယ်။

Diagram

```text
QR Found

↓

Popup

↓

Wait

1.5 sec

↓

Continue
```

---

# Step 17 — FPS Control

Loop အဆုံးမှာ

```java
Thread.sleep(40);
```

ဒီ Code က

CPU Usage

လျော့ပေးတယ်။

---

ဘာဖြစ်မလဲ

မရေးရင်?

```text
while(true){

}
```

CPU

100%

ဖြစ်သွားနိုင်တယ်။

---

40ms

Sleep

လုပ်လိုက်တော့

```text
Capture

↓

40ms Rest

↓

Capture

↓

40ms Rest
```

---

အကြမ်းဖျင်း

```text
1000 / 40

≈

25 FPS
```

---

# Complete Timeline

```text
Program Start

      │

      ▼

Create Thread

      │

      ▼

Find Camera

      │

      ▼

Open Camera

      │

      ▼

while(isRunning)

      │

      ▼

Capture Frame

      │

      ▼

Resize

      │

      ▼

Show GUI

      │

      ▼

Scan QR

      │

 ┌────┴─────┐

 │          │

No         Yes

 │          │

 ▼          ▼

Sleep     Verify

 │          │

 ▼          ▼

Repeat   Wait 1.5s

            │

            ▼

         Repeat
```

---

# Senior Developer Analysis

ဒီ Webcam Loop က Professional Design Pattern တစ်ခုကို အသုံးပြုထားပါတယ်။

```
Camera Layer
      │
      ▼
Frame Capture
      │
      ▼
UI Rendering
      │
      ▼
QR Detection
      │
      ▼
Business Logic
```

Layer တစ်ခုချင်းစီမှာ တာဝန်ကွဲကွဲရှိတယ်။

- **Capture** → `webcam.getImage()`
    
- **Render** → `lblCam.setIcon(...)`
    
- **Detect** → `decodeQRCode(image)`
    
- **Business Logic** → `onQrCodeScanned(qrData)`
    

ဒီလို Separation of Responsibilities လုပ်ထားတာကြောင့် Code ကို နောက်ပိုင်း Maintain လုပ်ရတာ လွယ်ကူပြီး Feature အသစ်ထည့်ရင်လည်း အပိုင်းလိုက် တိုးချဲ့နိုင်ပါတယ်။

---

# Chapter Summary

ဒီ Chapter မှာ သင်လေ့လာခဲ့တာတွေက

- ✅ `Webcam.getDefault()` နဲ့ Default Camera ကို ရယူတယ်။
    
- ✅ `webcam.open()` နဲ့ Camera ကို ဖွင့်တယ်။
    
- ✅ `while(isRunning)` က Scanner ရဲ့ Main Loop ဖြစ်တယ်။
    
- ✅ `webcam.getImage()` က Live Video Frame တစ်ခုချင်းစီကို `BufferedImage` အဖြစ် ရယူတယ်။
    
- ✅ Frame ကို Resize လုပ်ပြီး `SwingUtilities.invokeLater()` နဲ့ GUI ကို Update လုပ်တယ်။
    
- ✅ `decodeQRCode()` နဲ့ Frame တစ်ခုချင်းစီကို Scan လုပ်တယ်။
    
- ✅ QR တွေ့ရင် `onQrCodeScanned()` ကို ခေါ်ပြီး Verification Process စတင်တယ်။
    
- ✅ `Thread.sleep(40)` နဲ့ Frame Rate ကို ထိန်းပြီး CPU Usage ကို လျှော့ချပေးတယ်။
    
- ✅ `Thread.sleep(1500)` နဲ့ QR တစ်ခုကို အကြိမ်ကြိမ် Scan မဖြစ်အောင် ခဏစောင့်ပေးတယ်။
    

ဒီ Webcam Loop ဟာ QR Scanner Application တစ်ခုရဲ့ Core Engine ဖြစ်ပြီး Camera Capture, Live Preview, QR Detection နဲ့ Verification Process တွေကို ချိတ်ဆက်ပေးတဲ့ အဓိက Logic ဖြစ်ပါတယ်။