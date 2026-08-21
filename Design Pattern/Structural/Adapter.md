## 🔹 Adapter Pattern

### ပြဿနာက ဘာလဲ

Interface **မတူတဲ့** class နှစ်ခုကို အတူတကွ အလုပ်လုပ်စေချင်တဲ့အခါ ဖြစ်ပါတယ်။ ဥပမာ — third-party library တစ်ခုက ကိုယ့် system ရဲ့ interface နဲ့ **မကိုက်ညီဘူး**၊ ဒါပေမယ့် library ရဲ့ source code ကို **မပြင်နိုင်ဘူး** (jar file ပဲ ရှိလို့ ဒါမှမဟုတ် third-party ဖြစ်နေလို့)။

ဥပမာ — Application က `MediaPlayer` interface (`.mp3` play ရုံပဲ support) သုံးပေမယ့်၊ `.mp4` support ပေးတဲ့ third-party library (`AdvancedMediaPlayer`) ကို သုံးချင်တယ် ဆိုပါစို့။ Interface မတူလို့ တိုက်ရိုက် plug in မလုပ်နိုင်ဘူး။

### Adapter က ဘယ်လို ဖြေရှင်းလဲ

Class နှစ်ခုကြားမှာ **"ကူးတို" wrapper class** တစ်ခု ထားပါတယ် — client က မျှော်လင့်ထားတဲ့ interface ကို implement လုပ်ပြီး၊ အတွင်းထဲမှာတော့ incompatible class ကိုပဲ ခေါ်သုံးပါတယ် (electrical plug adapter နဲ့ အတူတူပါပဲ — plug shape ကွာရင် adapter သုံးသလိုမျိုး)။

### Java Code

```java
// Client က မျှော်လင့်ထားတဲ့ interface
interface MediaPlayer {
    void play(String filename);
}
```

```java
// Incompatible class - interface မတူဘူး, ပြင်လို့လည်းမရဘူး (third-party ဆိုထားပါစို့)
class AdvancedMediaPlayer {
    public void playMp4(String filename) {
        System.out.println("Playing mp4 file: " + filename);
    }
}
```

```java
// ⭐ Adapter - interface နှစ်ခုကြား ကူးတို
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter() {
        this.advancedPlayer = new AdvancedMediaPlayer();
    }

    @Override
    public void play(String filename) {
        // client ခေါ်တဲ့ play() ကို, incompatible class ရဲ့ method ဆီ ပြန်ညွှန်းတယ်
        advancedPlayer.playMp4(filename);
    }
}
```

```java
// Client class - MediaPlayer interface ကိုပဲ သိတယ်
class AudioPlayer implements MediaPlayer {
    private MediaAdapter adapter;

    @Override
    public void play(String filename) {
        if (filename.endsWith(".mp3")) {
            System.out.println("Playing mp3 file: " + filename);
        } else if (filename.endsWith(".mp4")) {
            adapter = new MediaAdapter(); // adapter ကနေတဆင့် ခေါ်တယ်
            adapter.play(filename);
        } else {
            System.out.println("Unsupported format: " + filename);
        }
    }
}
```

သုံးပုံ -

```java
AudioPlayer player = new AudioPlayer();
player.play("song.mp3"); // Playing mp3 file: song.mp3
player.play("movie.mp4"); // Playing mp4 file: movie.mp4 (adapter ကနေတဆင့်)
```

`AudioPlayer` က `AdvancedMediaPlayer` ကို **တိုက်ရိုက် မသိပါဘူး** — `MediaAdapter` ကတဆင့်သာ ဆက်သွယ်ပါတယ်။

### Real-world Java ဥပမာများ

- `Arrays.asList()` — array ကို `List` interface နဲ့ ကိုက်ညီအောင် wrap
- `InputStreamReader` — `InputStream` (byte-based) ကို `Reader` (character-based) အဖြစ် adapt လုပ်တယ်

### Factory Pattern တွေနဲ့ ဘာကွာသလဲ

Factory patterns တွေက **object ဖန်တီးတာ** ကို ဖြေရှင်းပေမယ့်၊ Adapter က **object ရှိပြီးသား** ကို interface ကွဲနေတာကို ချိတ်ပေးတာဖြစ်ပါတယ် — creational ပြဿနာ မဟုတ်ဘဲ **structural** ပြဿနာပါ။

### သတိထားရမယ့်အချက် (Trade-offs)

- Code ထဲ complexity အနည်းငယ် တိုးလာနိုင်ပါတယ် (class အသစ်တစ်ခု ထပ်ရေးရလို့)
- System design အစကတည်းက interface တွေကို သင့်တော်အောင် စဉ်းစားထားရင် Adapter လိုအပ်မှုနည်းသွားနိုင်ပါတယ် — ဒါက "after-the-fact fix" ပုံစံသဘောရှိပါတယ်

---

**Structural Patterns** ထဲက ပထမဆုံး Adapter ပြီးပါပြီ။ ဆက်ပြီး **Decorator Pattern** ကို သင်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီအထိ pattern ၆ ခုအတွက် quiz လေး လုပ်ကြည့်ချင်ပါသလား။