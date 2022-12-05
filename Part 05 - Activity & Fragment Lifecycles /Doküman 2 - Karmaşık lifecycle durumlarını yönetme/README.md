# <a name="1"></a>Karmaşık lifecycle durumlarını yönetme

- [Lifecycle hatalarından kaçının](#a)
- [Android lifecycle kütüphanesini kullanın](#b)
- [Uygulama kapanması simülasyonu yapın ve onSaveInstanceState() kullanın](#c)
- [Konfigürasyon değişikliklerini keşfedin](#d)

Bu dökümanda, önceki dökümandan DessertClicker uygulamasını genişleteceksiniz. Bir arka plan zamanlayıcı ekleyecek, ardından uygulamayı [Android lifecycle kütüphanesini](https://developer.android.com/topic/libraries/architecture/lifecycle) kullanacak şekilde dönüştüreceksiniz.

![app](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/5f5803e64f578dd5.png)

## <a name="a"></a>Aşama 1 : Lifecycle hatalarından kaçının

Önceki dökümanda, çeşitli lifecycle callbackleri override ederek ve sistem bu callbackleri çağırdığında log'a kaydederek activity'yi ve fragment lifecyclelarını nasıl gözlemleyeceğinizi öğrendiniz. Bu aşamada, DessertClicker uygulamasında lifecycle görevlerini yönetmenin daha karmaşık bir örneğini keşfedeceksiniz. Her saniye, çalıştığı saniye sayısıyla birlikte bir log ifadesi yazdıran bir zamanlayıcı kullanacaksınız.

### Adım 1:  DessertTimer'ı kurun

1. Son dökümandan DessertClicker uygulamasını açın. (Uygulamaya sahip değilseniz, DessertClickerLogs'u buradan indirebilirsiniz.)
2. **Project** görünümünde **Java > com.example.android.dessertclicker**'ı genişletin ve `DessertTimer.kt`'yi açın. Şu anda tüm kodun yorumlandığına dikkat edin, bu nedenle uygulamanın bir parçası olarak çalışmayacaktır.
3. Editör penceresindeki tüm kodu seçin. **Code > Comment with Line Comment**'i seçin veya `Control+/` (Mac'te `Command+/`) tuşlarına basın. Bu komut, dosyadaki tüm kodun yorumunu kaldırır. (Android Studio, siz uygulamayı yeniden oluşturana kadar unresolved reference hataları gösterebilir.)
4. `DessertTimer` class'ının, zamanlayıcıyı başlatan ve durduran `startTimer()` ve `stopTimer()` öğelerini içerdiğine dikkat edin. `startTimer()` çalışırken, zamanlayıcı her saniye, sürenin çalıştığı toplam saniye sayısıyla birlikte bir log mesajı yazdırır. `stopTimer()` metodu, sırayla, zamanlayıcıyı ve log ifadelerini durdurur.

>**Not:** `DessertTimer` class'ı, ilişkili `Runnable` ve `Handler` classlarıyla birlikte zamanlayıcı için bir background thread kullanır. Bu döküman için bunları bilmenize gerek yok (yine de daha sonraki bir dökümanda threadler hakkında daha fazla bilgi edineceksiniz). İlgileniyorsanız daha fazla ayrıntı için [Processes and threads](https://developer.android.com/guide/components/processes-and-threads)'e bakın.

5.` MainActivity.kt`'yi açın. Class'ın en üstünde, `dessertsSold` değişkeninin hemen altında, zamanlayıcı için bir değişken ekleyin:

```kotlin

private lateinit var dessertTimer: DessertTimer

```

6. `onCreate()` öğesine gidin ve `setOnClickListener()` çağrısından hemen sonra yeni bir `DessertTimer` nesnesi oluşturun:

```kotlin

dessertTimer = DessertTimer()

```

Artık bir dessert timer nesnesine sahip olduğunuza göre, **yalnızca** activity ekrandayken çalışmasını sağlamak için zamanlayıcıyı nereden başlatmanız ve durdurmanız gerektiğini düşünün. Sonraki adımlarda birkaç seçeneğe bakacaksınzı.

### Adım 2: Zamanlayıcıyı başlatın ve durdurun

onStart() metodu, activity görünür hale gelmeden hemen önce çağrılır. onStop() metodu, activity görünür olmayı bıraktıktan sonra çağrılır. Bu gcallbackler, zamanlayıcının ne zaman başlatılacağı ve durdurulacağı konusunda iyi adaylar gibi görünüyor.

1. `MainActivity` class'ında, `onStart()` callback'inde zamanlayıcıyı başlatın:


```kotlin

override fun onStart() {
   super.onStart()
   dessertTimer.startTimer()

   Timber.i("onStart called")
}

```

2. `onStop()`'ta zamanlayıcıyı durdurun.

```kotlin

override fun onStop() {
   super.onStop()
   dessertTimer.stopTimer()

   Timber.i("onStop Called")
}

```

3. Uygulamayı derleyin ve çalıştırın. Android Studio'da **Logcat** bölmesine tıklayın. Logcat arama kutusuna, hem `MainActivity` hem de `DessertTimer` classlarına göre filtre uygulayacak `dessertclicker` yazın. Uygulama başladığında, zamanlayıcının da hemen çalışmaya başladığına dikkat edin.

![Logcat](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/a399e0fea061361e.png)

4. **Geri** butonuna tıklayın ve zamanlayıcının tekrar durduğuna dikkat edin. Zamanlayıcı durur çünkü hem activity hem de kontrol ettiği zamanlayıcı yok edilmiştir.
5. Uygulamaya geri dönmek için recents ekranını kullanın. Logcat'te zamanlayıcının 0'dan yeniden başladığına dikkat edin.
6. **Paylaş** düğmesini tıklayın. Logcat'te zamanlayıcının hala çalıştığına dikkat edin.

![share button](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/e2319779260eb5ee.png)

7. **Home** butonuna tıklayın. Logcat'te zamanlayıcının çalışmayı durdurduğuna dikkat edin.
8. Uygulamaya geri dönmek için recents ekranını kullanın. Logcat'te, `onStart()` metodunda `startTimer()` öğesini çağırdığımız için zamanlayıcının kaldığı yerden yeniden başladığına dikkat edin.
9. `MainActivity`'de, `onStop()` metodunda `stopTimer()` çağrısını yorumlayın. `stopTimer()`'ın yorumlanması, `onStart()`'ta bir işlemi başlattığınız, ancak `onStop()`'ta tekrar durdurmayı unuttuğunuz durumu gösterir.
10. Uygulamayı derleyin ve çalıştırın ve zamanlayıcı başladıktan sonra Home butonuna tıklayın. Uygulama arka planda olmasına rağmen zamanlayıcı çalışıyor ve sürekli olarak sistem kaynaklarını kullanır. Zamanlayıcının devam etmesi, telefonunuzdaki bilgi işlem kaynaklarını gereksiz yere kullanabilir ve muhtemelen istediğiniz davranışı kullanamayabilir.

Genel pattern, bir callback'de bir şey kurduğunuzda veya başlattığınızda, karşılık gelen bir callback'de o şeyi durdurur veya kaldırırsınız. Bu şekilde, artık ihtiyaç duyulmadığında herhangi bir şeyin çalışmasını önlersiniz.

>**Not:** Müzik çalma gibi bazı durumlarda, şeyi çalışır durumda tutmak istersiniz. Bir şeyi çalışır durumda tutmanın uygun ve etkili yolları vardır, ancak bunlar bu dersin kapsamı dışındadır.

11. Zamanlayıcıyı durdurduğunuz yerde `onStop()`'daki yorumun kaldırın.
12. `onStart()` öğesinden `onCreate()` öğesine `startTimer()` çağrısını kesip yapıştırın. Bu değişiklik, bir kaynağı initialize etmek için `onCreate()` ve başlatmak için `onStart()` kullanmak yerine `onCreate()` içinde bir kaynağı hem initialize ettiğiniz hem de başlattığınız durumu gösterir.
13. Uygulamayı derleyin ve çalıştırın. Beklediğiniz gibi zamanlayıcının çalışmaya başladığına dikkat edin.
14. Uygulamayı durdurmak için Home'a tıklayın. Zamanlayıcı, beklediğiniz gibi çalışmayı durdurur.
15. Uygulamaya geri dönmek için recents ekranını kullanın. Bu durumda zamanlayıcının yeniden başlamadığına dikkat edin, çünkü `onCreate()` yalnızca uygulama başladığında çağrılır—bir uygulama ön plana döndüğünde çağrılmaz.

Hatırlanması gereken önemli noktalar:

- Bir lifecycle callback'inde bir kaynak kurduğunuzda (setup), kaynağı da parçalayın (teardown).
- İlgili metotlarda setup ve teardown yapın.
- `onStart()`'ta bir şey setup ederseniz, durdurun veya `onStop()`'ta tekrar teardown edin.

![lifecycle diagram](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/f6b25a71cec4e401.png)

## <a name="b"></a>Aşama 2 : Android lifecycle kütüphanesini kullanın

DessertClicker uygulamasında, zamanlayıcıyı `onStart()` içinde başlattıysanız, zamanlayıcıyı `onStop()` içinde durdurmanız gerektiğini görmek oldukça kolaydır. Yalnızca bir zamanlayıcı vardır, bu nedenle zamanlayıcıyı durdurmayı hatırlamak zor değildir.

Daha karmaşık bir Android uygulamasında, `onStart()` veya `onCreate()` içinde birçok şey ayarlayabilir, ardından hepsini `onStop()` veya `onDestroy()` içinde parçalayabilirsiniz. Örneğin, hem set up etmeniz hem de tear down etmeniz ve başlatmanız ve durdurmanız gereken animasyonlar, müzik, sensörler veya zamanlayıcılar olabilir. Birini unutursanız, bu hatalara ve baş ağrılarına yol açar.

[Android Jetpack](https://developer.android.com/jetpack)'in bir parçası olan _lifecycle kütüphanesi_ bu görevi basitleştirir. Kütüphane, bazıları farklı lifecycle durumlarında olan birçok hareketli parçayı izlemeniz gereken durumlarda özellikle yararlıdır. Kütüphane, lifecycleların çalışma şeklini değiştirir: Genellikle activity veya fragment, bir component'a (`DessertTimer` gibi) bir lifecycle callback'i gerçekleştiğinde ne yapacağını söyler. Ancak lifecycle kütüphanesini kullandığınızda, component'ın kendisi lifecycle değişikliklerini izler ve bu değişiklikler gerçekleştiğinde gerekeni yapar.

Lifecyle kütüphanesinin üç ana bölümü vardır:

- Bir lifecyle sahip olan componentlar olan lifecycle _ownerları_. `Activity` ve `Fragment`, lifecycle ownerlarıdır. Lifecycle ownerları, `LifecycleOwner` interface'ini uygular.
- Bir lifecycle ownerının gerçek durumunu tutan ve lifecycle değişiklikleri gerçekleştiğinde olayları tetikleyen `Lifecycle` class'ı.
- Lifecycle state'ini gözlemleyen ve lifecycle değiştiğinde görevleri gerçekleştiren lifecycle _observerları_. Lifecycle observerları, `LifecycleObserver` interface'ini uygular.
 
Bu aşamada, Android lifecyle kütüphanesini kullanmak için DessertClicker uygulamasını dönüştürecek ve kütüphanenin Android activity ve fragment lifecylelarıyla çalışmayı ve yönetmeyi nasıl kolaylaştırdığını öğreneceksiniz.
 
### Adım 1: DessertTimer'ı LifecycleObserver'a dönüştürün
 
Lifecyle kütüphanesinin önemli bir parçası, lifecyle gözlemi (observation) kavramıdır. Observation, classların (`DessertTimer` gibi) activity veya fragment lifecyle'ı hakkında bilgi sahibi olmasını ve bu lifecyle statelerindeki değişikliklere yanıt olarak kendilerini başlatıp durdurmasını sağlar. Bir lifecyle observer ile, activity ve fragment metotlarından nesneleri başlatma ve durdurma sorumluluğunu kaldırabilirsiniz.
 
1. `DesertTimer.kt` sınıfını açın.
2. `DessertTimer` class sınıf imzasını şöyle görünecek şekilde değiştirin:
 
```kotlin

class DessertTimer(lifecycle: Lifecycle) : LifecycleObserver {

``` 
 
Bu yeni class tanımı iki şey yapar:

- Constructor, zamanlayıcının gözlemlediği lifecycle olan bir `Lifecycle` nesnesi alır.
- Class tanımı, `LifecycleObserver` interface'ini uygular.

3. `DessertTimer` classındaki `runnable` değişkenin bildiriminin altına, class tanımına bir `init` bloğu ekleyin. `init` bloğunda, owner'ından (activity) geçirilen lifecycle nesnesini bu class'a (observer) bağlamak için `addObserver()` metodunu kullanın.

```kotlin

init {
   lifecycle.addObserver(this)
}

``` 

4. `@OnLifecycleEvent` ile `startTimer()`'a annotation ekleyin ve `ON_START` lifecycle event'ini kullanın. Lifecycle observer'ınızın gözlemleyebileceği tüm lifecyle eventleri [`Lifecycle.Event`](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle.Event.html) class'ındadır. Örneğin, `@OnLifecycleEvent(Lifecycle.Event.ON_START)` annotation'ı, aşağıdaki metodun `onStart` lifecyle eventlerini izlediğini belirtir.

```kotlin

@OnLifecycleEvent(Lifecycle.Event.ON_START)
fun startTimer() {

``` 

5. `ON_STOP` event'ini kullanarak `stopTimer()` için aynı şeyi yapın:

```kotlin

@OnLifecycleEvent(Lifecycle.Event.ON_STOP)
fun stopTimer()

``` 

### Adım 2: MainActivity'yi değiştir

`MainActivity` class'ınız, object-oriented inheritance yoluyla zaten bir lifecycle sahibidir. `AppCompatActivity`'den `MainActivity` subclasslarına dikkat edin, bu da `FragmentActivity`'den subclasslar oluşturur. `FragmentActivity` superclass'ı `LifecycleOwner`'ı uyguladığından, activity'nizin lifecycle'ından haberdar olmasını sağlamak için yapmanız gereken başka bir şey yoktur. Tek yapmanız gereken activity'nin lifecycle nesnesini `DessertTimer` constructor'a iletmektir.

1. `MainActivity`'yi açın. `onCreate()` metodunda, `DessertTimer`'ın initialization'ını `this.lifecycle`'ı içerecek şekilde değiştirin:
 
```kotlin

dessertTimer = DessertTimer(this.lifecycle)

```  
 
Activity'nin `lifecycle` özelliği, bu activity'nin sahip olduğu `Lifecycle` nesnesini tutar.

2. `onCreate()` içindeki `startTimer()` çağrısını ve `onStop()` içindeki `stopTimer()` çağrısını kaldırın. Artık activity'de ne yapacağını `DessertTimer`'a söylemenize gerek yok, çünkü `DessertTimer` artık lifecycle'ı gözlemliyor ve lifecycle state'i değiştiğinde otomatik olarak bilgilendiriliyor. Bu callbacklerde artık tek yaptığınız bir mesaj kaydetmek.
3. Uygulamayı derleyin ve çalıştırın ve Logcat'i açın. Zamanlayıcının beklendiği gibi çalışmaya başladığına dikkat edin.
 
![Logcat](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/a399e0fea061361e.png)
 
4. Uygulamayı arka plana koymak için home düğmesini tıklayın. Zamanlayıcının beklendiği gibi çalışmayı durdurduğuna dikkat edin.
 
## <a name="c"></a>Aşama 3 : Uygulama kapanması simülasyonu yapın ve onSaveInstanceState() kullanın

Android bu uygulamayı arka plandayken kapatırsa uygulamanıza ve verilerine ne olur? Bu zor vakayı anlamak önemlidir.

Uygulamanız arka plana geçtiğinde yok edilmez, yalnızca durdurulur ve kullanıcının ona dönmesini bekler. Ancak Android işletim sisteminin ana endişelerinden biri, ön planda olan etkinliğin sorunsuz çalışmasını sağlamaktır. Örneğin, kullanıcınız bir otobüsü yakalamalarına yardımcı olmak için bir GPS uygulaması kullanıyorsa, bu GPS uygulamasını hızlı bir şekilde oluşturmanız ve yol tariflerini göstermeye devam etmeniz önemlidir. Kullanıcının birkaç gün bakmamış olabileceği DessertClicker uygulamasının arka planda sorunsuz çalışır durumda kalması daha az önemlidir.

Android, arka plan uygulamalarını, ön plan uygulamasının sorunsuz çalışabilmesi için düzenler. Örneğin, Android, arka planda çalışan uygulamaların yapabileceği işlem miktarını sınırlar.

Bazen Android, uygulamayla ilişkili her activity'yi içeren tüm uygulama sürecini bile kapatır. Android, sistem stresli olduğunda ve görsel olarak gecikme tehlikesi olduğunda bu tür bir kapatma yapar, bu nedenle bu noktada ek bir callbackleri veya kod çalıştırılmaz. Uygulamanızın süreci arka planda sessizce kapanır. Ancak kullanıcıya, uygulama kapatılmış gibi görünmüyor. Kullanıcı, Android işletim sisteminin kapattığı bir uygulamaya geri döndüğünde, Android o uygulamayı yeniden başlatır.

Bu aşamada, bir Android işleminin kapanmasını simüle edecek ve tekrar başladığında uygulamanıza ne olduğunu inceleyebilirsiniz.

>**Not:** Başlamadan önce, API 28 veya üstünü destekleyen bir emülatör veya cihaz çalıştırdığınızdan emin olun.

### Adım 1: Bir işlemin kapanmasını simüle etmek için adb kullanın

Android Debug Bridge (`adb`), bilgisayarınıza bağlı emülatörlere ve aygıtlara talimatlar göndermenize olanak tanıyan bir command-line aracıdır. Bu adımda, uygulamanızın process'ini kapatmak ve Android uygulamanızı kapattığında ne olduğunu görmek için `adb`'yi kullanırsınız.

1. Uygulamanızı derleyin ve çalıştırın. Cupcake üzerine birkaç kez tıklayın.
2. Uygulamanızı arka plana almak için Home düğmesine basın. Uygulamanız artık durdurulmuştur ve Android, uygulamanın kullandığı kaynaklara ihtiyaç duyarsa uygulama kapatılabilir.
3. Android Studio'da, command-line terminalini açmak için **Terminal** sekmesine tıklayın.

![Terminal](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/a37c3740cc57b67d.png)

4. `adb` yazın ve enter'a tıklayın.

`Android Debug Bridge version X.XX.X` ile başlayan ve `tags to be used by logcat (see logcat --help)` ile biten birçok çıktı görürseniz, her şey yolunda demektir. Bunun yerine `adb: command not found` görürseniz, execution path'inizde `adb` komutunun mevcut olduğundan emin olun. Talimatlar için, [Utilies bölümündeki](https://developer.android.com/courses/extras/utilities) "Add adb to your execution path" konusuna bakın.

5. Bu yorumu kopyalayıp komut satırına yapıştırın ve Return tuşuna basın:

```

adb shell am kill com.example.android.dessertclicker

```

Bu komut, bağlı tüm aygıtlara veya emülatörlere, yalnızca uygulama arka plandaysa, `dessertclicker` paket adıyla işlemi sonlandırmak için bir STOP mesajı göndermelerini söyler. Uygulamanız arka planda olduğundan, işleminizin durdurulduğunu gösteren cihaz veya emülatör ekranında hiçbir şey görünmez. Android Studio'da, çağrılan `onStop()` metodunu görmek için **Run** sekmesine tıklayın. `onDestroy()` geri çağrısının hiçbir zaman çalıştırılmadığını görmek için **Logcat** sekmesine tıklayın—activity'niz basitçe sona erdi.

6. Uygulamaya geri dönmek için recents ekranını kullanın. Uygulamanız, arka plana alınmış veya tamamen durdurulmuş olsun, sonlarda görünür. Uygulamaya dönmek için recents ekranını kullandığınızda activity yeniden başlatılır. Activity, `onCreate()` dahil olmak üzere tüm başlangıç lifecycle callbacklerinden geçer.
7. Uygulama yeniden başlatıldığında, "puanınızı" (hem satılan tatlı sayısı hem de toplam dolar) varsayılan değerlere (0) sıfırladığına dikkat edin. Android uygulamanızı kapattıysa, neden state'inizi kaydetmedi?

İşletim sistemi sizin için uygulamanızı yeniden başlattığında, Android, uygulamanızı önceki state'e sıfırlamak için elinden geleni yapar. Android, bazı viewlarınızın state'ini alır ve activity'den çıktığınızda bir bundle'da kaydeder. Otomatik olarak kaydedilen verilere bazı örnekler, bir EditText'teki metin (view'da ayarlanmış bir IDleri olduğu sürece) ve activity'nizin back stack'idir.

Ancak, bazen Android işletim sistemi tüm verilerinizi bilmez. Örneğin, DessertClicker uygulamasında `revenue` gibi özel bir değişkeniniz varsa, Android işletim sistemi bu verileri veya activity'niz için önemini bilmez. Bu verileri bundle'a kendiniz eklemeniz gerekir.

## Adım 2: Bundle verilerini kaydetmek için onSaveInstanceState() kullanın

`onSaveInstanceState()` metodu, Android işletim sistemi uygulamanızı yok ederse ihtiyaç duyabileceğiniz tüm verileri kaydetmek için kullandığınız callback'tir.Lifecycle callback şemasında, activity durdurulduktan sonra `onSaveInstanceState()` çağrılır. Uygulamanız her arka plana geçtiğinde çağrılır.

![lifecycle diagram](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/c259ab6beca0ca88.png)

`onSaveInstanceState()` çağrısını bir güvenlik önlemi olarak düşünün; etkinliğiniz ön plandan çıkarken size az miktarda bilgiyi bir bundle'a kaydetme şansı verir. Sistem bu verileri şimdi kaydeder çünkü uygulamanızı kapatana kadar beklerse işletim sistemi kaynak baskısı altında olabilir. Verilerin her seferinde kaydedilmesi, gerektiğinde bundle'daki güncelleme verilerinin geri yüklenebilmesini sağlar.

1. `MainActivity`'de `onSaveInstanceState()` callback'ini override edin ve bir `Timber` log ifadesi ekleyin.

```kotlin

override fun onSaveInstanceState(outState: Bundle) {
   super.onSaveInstanceState(outState)

   Timber.i("onSaveInstanceState Called")
}

```

>**Not:** `onSaveInstanceState()` için biri yalnızca bir `outState` parametresi olan ve diğeri `outState` ve `outPersistentState` parametrelerini içeren iki override vardır. Tek `outState` parametresiyle yukarıdaki kodda gösterileni kullanın.

2. Uygulamayı derleyin ve çalıştırın ve arka plana koymak için **Home** butonunu tıklayın. `onSaveInstanceState()` callbackleri hem `onPause()` hem de `onStop()`'tan hemen sonra gerçekleştiğine dikkat edin:

![Logcat](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/c80ed57c9e4e23a.png)

3. Dosyanın en üstünde, class tanımından hemen önce şu sabitleri ekleyin:

```kotlin

const val KEY_REVENUE = "revenue_key"
const val KEY_DESSERT_SOLD = "dessert_sold_key"
const val KEY_TIMER_SECONDS = "timer_seconds_key"

```

Bu key'leri, instance state bundle'ından hem kaydetmek hem de veri almak için kullanacaksınız.

4. `onSaveInstanceState()`'e ilerleyin ve `Bundle` türündeki `outState` parametresine dikkat edin.

Bundle, key'lerin her zaman string olduğu bir key-value (anahtar/değer) çiftleri koleksiyonudur. Bundle'a `int` ve `boolean` değerleri gibi primitive değerler koyabilirsiniz. Sistem bu bundle'ı RAM'de tuttuğundan, bundle'daki verileri küçük tutmak best practice'tir. Bu bundle!'ın boyutu da sınırlıdır, ancak boyut cihazdan cihaza değişir. Genellikle 100.000'den çok daha az depolamanız gerekir, aksi takdirde uygulamanızı `TransactionTooLargeException` hatasıyla çökertme riskiyle karşı karşıya kalırsınız.

5. `onSaveInstanceState()` içinde, `putInt()` metoduyla `revenue` değerini (bir tamsayı) bundle'a koyun:

```kotlin

outState.putInt(KEY_REVENUE, revenue)

```

`putInt()` metodu (ve `Bundle` sınıfından `putFloat()` ve `putString(`) gibi benzer metotlar) iki argüman alır: key için bir string (`KEY_REVENUE` sabiti) ve kaydedilecek gerçek value.

6. Satılan tatlı sayısı ve zamanlayıcının state'i ile aynı işlemi tekrarlayın:

```kotlin

outState.putInt(KEY_DESSERT_SOLD, dessertsSold)
outState.putInt(KEY_TIMER_SECONDS, dessertTimer.secondsCount)

```

### Adım 3: Bundle verilerini geri yüklemek için onCreate() kullanın

1. onCreate()'e ilerleyin ve metot imzasını inceleyin:

```kotlin

override fun onCreate(savedInstanceState: Bundle?) {

```

`onCreate()` her çağrıldığında bir `Bundle` aldığına dikkat edin. Bir işlemin kapanması nedeniyle activity'niz yeniden başlatıldığında, kaydettiğiniz bundle `onCreate()`'e iletilir. Activity'niz yeni başlıyorsa, `onCreate()` içindeki bu bundle `null`'dır. Bu nedenle, bundle `null` değilse, activity'yi önceden bilinen bir noktadan "yeniden yarattığınızı" bilirsiniz.

>**Not:** Activity yeniden yaratılıyorsa, `onRestoreInstanceState()` callback'i, yine bundle ile birlikte `onStart()`'tan sonra çağrılır. Çoğu zaman, activity state'ini `onCreate()` içinde geri yüklersiniz. Ancak `onRestoreInstanceState()` `onStart()`'tan sonra çağrıldığından, `onCreate()` çağrıldıktan sonra bir state'i geri yüklemeniz gerekirse, `onRestoreInstanceState()` öğesini kullanabilirsiniz.

2. Bu kodu, `DessertTimer` kurulumundan sonra `onCreate()` öğesine ekleyin:


```kotlin

if (savedInstanceState != null) {
   revenue = savedInstanceState.getInt(KEY_REVENUE, 0)
}

```

`Null` testi, bundle'da veri olup olmadığını veya bundle'ın `null` olup olmadığını belirler, bu da size uygulamanın yeni başlatılıp başlatılmadığını veya bir kapatmadan sonra yeniden oluşturulup oluşturulmadığını söyler. Bu test, bundle'dan verileri geri yüklemek için yaygın bir pattern'dır.

Burada kullandığınız key'in (`KEY_REVENUE`) `putInt()` için kullandığınız key ile aynı olduğuna dikkat edin. Her seferinde aynı key'i kullandığınızdan emin olmak için bu keyleri sabitler olarak tanımlamak best practice'tie. Verileri bundle'a koymak için `putInt()`'i kullandığınız gibi, veriyi bundle'dan çıkarmak için `getInt()`'i kullanırsınız. `getInt()` metodu iki argüman alır:

- Key görevi gören bir string, örneğin gelir (revenue) değeri için `"key_revenue"`.
- Bundle'da bu key için hiçbir değer olmaması durumunda varsayılan bir value.

Bundle'dan aldığınız integer daha sonra `revenue` değişkenine atanır ve UI bu değeri kullanır.

3. Satılan tatlıların sayısını ve zamanlayıcının değerini geri yüklemek için `getInt()` metotlarını ekleyin:

```kotlin

if (savedInstanceState != null) {
   revenue = savedInstanceState.getInt(KEY_REVENUE, 0)
   dessertsSold = savedInstanceState.getInt(KEY_DESSERT_SOLD, 0)
   dessertTimer.secondsCount =
       savedInstanceState.getInt(KEY_TIMER_SECONDS, 0)
}

```

4. Uygulamayı derleyin ve çalıştırın.
5. Android Studio menüsünden **Run > Stop 'app'** öğesini seçerek uygulamayı durdurun. Artık uygulama cihazınıza/emülatörünüze yüklenmiştir ancak uygulama çalışmıyor.
6. Cihazınızda/emülatörünüzde app launcher'ı açın ve **Dessert Clicker** uygulamasını seçin. Donut haline gelene kadar cupcake'e en az beş kez basın. Uygulamayı arka plana koymak için Home'a tıklayın.
7. Android Studio **Terminal** sekmesinde, uygulamanın sürecini kapatmak için `adb` komutunu çalıştırın.

```

adb shell am kill com.example.android.dessertclicker

```

8. Uygulamaya geri dönmek için recents ekranını kullanın. Bu sefer uygulamanın, bundle'dan doğru revenue ve desserts sold değerleri ile geri döndüğüne dikkat edin. Ama aynı zamanda tatlının bir cupcake döndüğüne de dikkat edin. Uygulamanın kapatıldığından tam olarak olduğu gibi dönmesini sağlamak için yapılacak bir şey daha var.
9. `MainActivity`'de `showCurrentDessert()` metodunu inceleyin. Bu metodun, satılan tatlıların mevcut sayısına ve `allDesserts` değişkenindeki tatlı listesine dayalı olarak activity'de hangi tatlı görüntüsünün görüntüleneceğini belirlediğine dikkat edin.

```kotlin

for (dessert in allDesserts) {
   if (dessertsSold >= dessert.startProductionAmount) {
       newDessert = dessert
   }
    else break
}

```

Bu metot, doğru görüntüyü seçmek için satılan tatlı sayısına dayanır. Bu nedenle, `onSaveInstanceState()` içindeki bundle'daki resme bir referansı saklamak için herhangi bir şey yapmanız gerekmez. Bu bundle'da, satılan tatlıların sayısını zaten saklıyorsunuz.

10. `OnCreate()` içinde, bundle'daki verileri geri yüklemek için `showCurrentDessert()` metodunu çağırın.

```kotlin

if (savedInstanceState != null) {
   revenue = savedInstanceState.getInt(KEY_REVENUE, 0)
   dessertsSold = savedInstanceState.getInt(KEY_DESSERT_SOLD, 0)
   dessertTimer.secondsCount = 
      savedInstanceState.getInt(KEY_TIMER_SECONDS, 0)
   showCurrentDessert()                   
}

```

11. Uygulamayı derleyin ve çalıştırın.
12. Android Studio menüsünden **Run > Stop 'app'** öğesini seçerek uygulamayı durdurun. Artık uygulama cihazınıza/emülatörünüze yüklenmiştir ancak uygulama çalışmıyor.
13. App launcher'ı açın ve **Dessert Clicker** uygulamasını seçin. Tatlı simgesine birkaç kez tıklayın.
14. Uygulamayı arka plana alın. İşlemi kapatmak için `adb` kullanın.

```

adb shell am kill com.example.android.dessertclicker

```

15. Uygulamaya geri dönmek için recents ekranını kullanın. Hem anlatılan tatlılar, toplam gelir hem de tatlı görüntüsü için değerlerin doğru şekilde geri yüklendiğini unutmayın.

## <a name="d"></a>Aşama 4 : Konfigürasyon değişikliklerini keşfedin

Activity ve fragment lifecycle'ı yönetmede anlaşılması önemli olan son bir özel durum vardır: konfigürasyon değişikliklerinin activitylerinizin ve fragmentlarınızın lifecycle'ını nasıl etkilediği.

Bir _konfigürasyon değişikliği_, cihazın durumu o kadar radikal bir şekilde değiştiğinde meydana gelir ki, sistemin değişikliği çözmesinin en kolay yolu activity'yi tamamen kapatmak ve yeniden oluşturmaktır. Örneğin, kullanıcı aygıt dilini değiştirirse, farklı metin yönlerine uyum sağlamak için tüm layout'un değişmesi gerekebilir. Kullanıcı cihazı bir dock'a takarsa veya fiziksel bir klavye eklerse, uygulama layout'unun farklı bir ekran boyutu veya layout'undan faydalanması gerekebilir. Aygıt yönü değişirse, örneğin aygıt dikeyden yataya veya diğer yöne döndürülürse, düzenin yeni yönelime uyacak şekilde değiştirilmesi gerekebilir.

### Adım 1: Cihaz rotasyonunu ve lifecycle callbacklerini keşfedin

1. Uygulamanızı derleyip çalıştırın ve Logcat'i açın.
2. Cihazı veya emülatörü yatay moda çevirin. Emülatörü döndürme düğmeleriyle veya `Control` ve ok (→) tuşlarıyla (Mac'te `Command` ve ok (→) tuşları) sola veya sağa döndürebilirsiniz.

![emulator](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/623fce7c623d42bd.png)

3. Çıktıyı Logcat'te inceleyin. Çıktıyı `MainActivity` ile filtreleyin.

![Logcat](https://developer.android.com/codelabs/kotlin-android-training-complex-lifecycle/img/a9e19f1f0f43fffe.png)

Aygıt veya emülatör ekranı döndürdüğünde, sistemin activity'yi kapatmak için tüm lifecycle callbacklerini çağırdığına dikkat edin. Ardından, activity yeniden oluşturulurken sistem, activity'yi başlatmak için tüm lifecycle callbacklerini çağırır.

4. `MainActivity`'de, `onSaveInstanceState()` metodunun tamamını yorumlayın.
5. Uygulamanızı tekrar derleyin ve çalıştırın. Cupcake'e birkaç kez tıklayın ve cihazı veya emülatörü döndürün. Bu sefer cihaz döndürüldüğünde ve activity kapatılıp yeniden oluşturulduğunda activity varsayılan değerlerle başlar.

Bir konfigürasyon değişikliği meydana geldiğinde Android, uygulamanın state'ini kaydetmek ve geri yüklemek için önceki aşamada öğrendiğiniz aynı instance state bundle'ı kullanır. İşlem kapatmada (process shutdown) olduğu gibi, uygulamanızın verilerini bundle'a koymak için `onSaveInstanceState()` kullanın. Ardından, cihaz döndürülürse activity state'i verilerini kaybetmemek için `onCreate()` içindeki verileri geri yükleyin.

6. `MainActivity`'de `onSaveInstanceState()` metodunun yorumunu kaldırın, uygulamayı çalıştırın, cupcake'e tıklayın ve uygulamayı veya cihazı döndürün. Bu sefer tatlı verilerinin activity rotasyonu boyunca korunduğuna dikkat edin.
