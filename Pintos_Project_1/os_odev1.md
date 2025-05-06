---- GRUP ----

>> Grup üyelerinizin adlarını ve e-posta adreslerini doldurun.

BERAT DİZDAR <2023280103@ogr.deu.edu.tr>  
HAMED SHER MOHAMMAD ISHAQ <2023280048@ogr.deu.edu.tr>  

---- ÖN HAZIRLIKLAR ----

>> Teslimatınızla ilgili herhangi bir ön hazırlık yorumunuz varsa lütfen burada belirtin.

Projede bayağı zorlandık ama sonunda çalışır hale getirdik. Özellikle priority donation kısmında çok vakit harcadık. Kodda /* Solution Code */ şeklinde işaretledik bizim eklediğimiz kısımları.

>> Pintos dokümantasyonu, ders kitabı, ders notları dışında, teslimatınızı hazırlarken başvurduğunuz çevrimdışı veya çevrimiçi kaynakları belirtiniz.

1- chatgpt.com
2- chat.deepseek.com
3- Youtube :
    - https://youtube.com/playlist?list=PLh9ECzBB8tJO9eiwfQbcA2ThMbUSkbOWf&si=5nGolSbefqY9-fQP
    - https://youtube.com/playlist?list=PLmQBKYly8OsWiRYGn1wvjwAdbuNWOBJNf&si=100NPqyR6gzY3qh6
    - https://youtube.com/playlist?list=PLPQ7PivebcX5RR5D6OSNFum3DxaL9g7Wh&si=h4n8FMtw2MFD_Ryh
    - https://youtube.com/playlist?list=PLOZTHmYuABf0rONeeOdvTnFDKF_7DDByy&si=rbHJdzxtAk8rdXNG
    - https://www.youtube.com/@CoreDumpped
4- diger siteler :
    - stackoverflow.com
    - geeksforgeeks.org
    - github.com

                 ALARM SAATİ
                 ===========

---- VERİ YAPILARI ----

>> A1: Her yeni veya değiştirilen `struct` veya `struct` üyesi, global veya statik değişken, `typedef` veya enumerasyonun beyanını buraya kopyalayın. Her birinin amacını 25 kelime veya daha az bir şekilde tanımlayın.

```c
/* thread.h içinde struct thread'e ekledik: */
int64_t ticks_blocked;              /* Thread'in kaç tick bloke kalacağını tutar */
```

Thread'in ne kadar süre uyuyacağını takip etmek için bu değişkeni ekledik. timer_sleep() çağrıldığında bu değeri ayarlıyoruz, sonra timer kesme işleyicisinde her tick'te azaltıyoruz.

---- ALGORİTMALAR ----

>> A2: timer_sleep() fonksiyonuna yapılan bir çağrıda neler olduğunu, zamanlayıcı kesme işleyicisinin etkilerini kısaca açıklayın.

timer_sleep() çağrıldığında:

1. Önce ticks değerinin 0 veya negatif olup olmadığını kontrol ediyoruz, öyleyse hemen dönüyoruz.
2. Kesmeleri kapatıyoruz ki işlem sırasında başka bir şey araya girmesin.
3. Çalışan thread'in ticks_blocked değişkenine parametre olarak gelen ticks değerini atıyoruz.
4. thread_block() ile thread'i bloke ediyoruz.
5. Kesmeleri tekrar açıyoruz.

Zamanlayıcı kesme işleyicisinde:
1. thread_foreach() fonksiyonu ile tüm thread'leri dolaşıp checkInvoke() fonksiyonunu çağırıyoruz.
2. checkInvoke() içinde, eğer thread bloke durumundaysa ve ticks_blocked değeri pozitifse, bu değeri azaltıyoruz.
3. Eğer ticks_blocked sıfıra ulaşırsa, thread_unblock() ile thread'i tekrar hazır duruma getiriyoruz.

Bu şekilde busy-waiting yapmadan thread'leri uyutabiliyoruz.

>> A3: Zamanlayıcı kesme işleyicisi içinde harcanan süreyi minimize etmek için hangi adımlar atılmaktadır?

Kesme işleyicisinde harcanan süreyi azaltmak için:

1. checkInvoke() fonksiyonunu olabildiğince basit tuttuk, sadece gerekli işlemleri yapıyor.
2. Thread'i sadece ticks_blocked sıfır olduğunda unblock ediyoruz, her seferinde kontrol etmiyoruz.
3. Karmaşık hesaplamalar yapmıyoruz, sadece basit bir sayaç azaltma işlemi var.
4. Gereksiz kontroller eklemedik, sadece bloke thread'ler için işlem yapıyoruz.
5. Ayrı bir uyuyan thread listesi tutmadık, bu da ek liste işlemlerinden kaçınmamızı sağladı.

Aslında daha verimli olması için ayrı bir uyuyan thread listesi tutmayı düşündük ama bu sefer de liste işlemleri için ek yük olacaktı, bu yüzden mevcut yapıyı kullanmaya karar verdik.

---- SENKRONİZASYON ----

>> A4: Birden fazla iş parçacığı aynı anda timer_sleep() fonksiyonunu çağırdığında yarış koşulları nasıl engellenir?

Birden fazla thread aynı anda timer_sleep() çağırdığında yarış koşullarını engellemek için:

1. timer_sleep() içinde intr_disable() ile kesmeleri kapatıyoruz. Bu sayede fonksiyonun kritik bölümü kesintiye uğramadan çalışıyor.
2. Thread'in ticks_blocked değerini ayarlama ve thread'i bloke etme işlemlerini atomik olarak yapıyoruz.
3. İşlem bittikten sonra intr_set_level() ile kesmeleri tekrar açıyoruz.

Bu şekilde, birden fazla thread aynı anda timer_sleep() çağırsa bile, her biri sırayla işlem görüyor ve birbirlerinin işlemlerini bozmuyorlar.

>> A5: timer_sleep() fonksiyonu çağrıldığında bir zamanlayıcı kesmesi meydana gelirse yarış koşulları nasıl engellenir?

timer_sleep() çalışırken zamanlayıcı kesmesi olursa:

1. Fonksiyonun başında intr_disable() ile kesmeleri kapatıyoruz, böylece kritik bölüm çalışırken kesme oluşmuyor.
2. Kesme ancak kesmeleri kapattığımız satırdan önce veya açtığımız satırdan sonra oluşabilir.
3. Eğer kesme, kesmeleri kapatmadan önce olursa, henüz thread'in durumunu değiştirmediğimiz için sorun olmaz.
4. Eğer kesme, kesmeleri açtıktan sonra olursa, thread zaten bloke edilmiş durumda olduğu için sorun olmaz.

Yani intr_disable() ve intr_set_level() kullanarak kritik bölümü koruyoruz ve yarış koşullarını engelliyoruz.

---- GEREKÇE ----

>> A6: Bu tasarımı neden seçtiniz? Diğer düşündüğünüz tasarımlara kıyasla hangi açılardan daha üstün olduğunu düşünüyorsunuz?

Bu tasarımı seçtik çünkü:

1. Basit ve anlaşılır: Sadece bir değişken ekleyip, onu azaltarak işi hallediyoruz.
2. Verimli: Busy-waiting yapmıyoruz, CPU zamanını boşa harcamıyoruz.
3. Kolay uygulanabilir: Mevcut thread yapısına minimal değişiklik yaparak çözüm ürettik.

Düşündüğümüz diğer tasarım, uyuyan thread'leri ayrı bir listede tutmaktı. Bu tasarımda:
- Uyanma zamanına göre sıralanmış bir liste tutacaktık
- Her tick'te sadece listenin başındaki elemanları kontrol edecektik
- Bu şekilde tüm thread'leri dolaşmak zorunda kalmayacaktık

Ama bu tasarımın dezavantajları vardı:
- Liste işlemleri için ek yük gerekecekti
- Sıralı liste tutmak ve güncellemek daha karmaşık olacaktı
- Mevcut thread yönetim sistemine daha fazla değişiklik yapmak gerekecekti

Sonuçta, mevcut tasarımımız daha basit, daha az hata riski taşıyor ve yeterince verimli çalışıyor. Bu yüzden bu tasarımı tercih ettik.

             ÖNCELİKLE PLANLAMA
             ===================

---- VERİ YAPILARI ----

>> B1: Her yeni veya değiştirilen `struct` veya `struct` üyesi, global veya statik değişken, `typedef` veya enumerasyonun beyanını buraya kopyalayın. Her birinin amacını 25 kelime veya daha az bir şekilde tanımlayın.

```c
/* thread.h içinde struct thread'e ekledik: */
int base_priority;                  /* Thread'in orijinal önceliği */
struct list locks_holding;          /* Thread'in tuttuğu kilitler listesi */
struct lock *lock_waiting4;         /* Thread'in beklediği kilit */

/* synch.h içinde struct lock'a ekledik: */
struct list_elem elem;              /* Kilit listesi elemanı */
int max_priority;                   /* Kilidi bekleyen en yüksek öncelikli thread'in önceliği */
```

base_priority: Thread'in donation olmadan önceki gerçek önceliğini saklıyoruz.
locks_holding: Thread'in hangi kilitleri tuttuğunu takip etmek için kullanıyoruz.
lock_waiting4: Thread'in hangi kilidi beklediğini gösteriyor, nested donation için gerekli.
elem: Kilidi thread'in locks_holding listesine eklemek için kullanıyoruz.
max_priority: Kilidi bekleyen en yüksek öncelikli thread'in önceliğini tutuyoruz.

>> B2: Öncelik bağışını izlemek için kullanılan veri yapısını açıklayın. Bir iç içe bağışı ASCII sanatı ile diyagramlayın. (Alternatif olarak, bir .png dosyası gönderin.)

Öncelik bağışını izlemek için yukarıdaki veri yapılarını kullanıyoruz. Bir thread bir kilit için beklediğinde, o kilidin sahibine önceliğini bağışlıyor. Eğer kilit sahibi de başka bir kilit için bekliyorsa, bağışlanan öncelik o kilidin sahibine de geçiyor.

İç içe bağış örneği:

```
Thread H (öncelik 63)
    |
    | bekliyor
    V
Lock A (max_priority 63)
    |
    | sahibi
    V
Thread M (base_priority 31, gerçek öncelik 63)
    |
    | bekliyor
    V
Lock B (max_priority 63)
    |
    | sahibi
    V
Thread L (base_priority 0, gerçek öncelik 63)
```

Bu örnekte:
- Yüksek öncelikli H thread'i (63) Lock A'yı bekliyor
- Lock A, orta öncelikli M thread'i (31) tarafından tutuluyor
- H, önceliğini M'ye bağışlıyor, M'nin önceliği 63 oluyor
- M thread'i Lock B'yi bekliyor
- Lock B, düşük öncelikli L thread'i (0) tarafından tutuluyor
- M, bağışlanan önceliğini L'ye bağışlıyor, L'nin önceliği 63 oluyor

Bu şekilde öncelik bağışı zinciri oluşuyor ve priority inversion problemi çözülüyor.

---- ALGORİTMALAR ----

>> B3: Bir kilit, semafor veya koşul değişkeni için bekleyen en yüksek öncelikli iş parçacığının ilk önce uyanmasını nasıl sağlarsınız?

Bekleyen en yüksek öncelikli thread'in ilk uyanmasını sağlamak için:

Kilitler ve semaforlar için:
1. Semafor'un waiters listesine thread'leri eklerken list_insert_ordered() kullanıyoruz, böylece liste önceliğe göre sıralı oluyor.
2. sema_down() içinde thread'i waiters listesine eklerken thread_cmp_priority fonksiyonunu kullanıyoruz.
3. sema_up() içinde, thread'i uyandırmadan önce waiters listesini tekrar sıralıyoruz (list_sort), çünkü beklerken öncelikler değişmiş olabilir.
4. Sonra listenin başındaki (en yüksek öncelikli) thread'i uyandırıyoruz.

Koşul değişkenleri için:
1. cond_cmp_priority() adında özel bir karşılaştırma fonksiyonu yazdık.
2. cond_signal() içinde, waiters listesini bu fonksiyonla sıralıyoruz.
3. Böylece en yüksek öncelikli thread'in semaforu ilk sinyallendirilmiş oluyor.

Bu şekilde her zaman en yüksek öncelikli thread'in ilk uyanmasını garanti ediyoruz.

>> B4: lock_acquire() fonksiyonu bir öncelik bağışı oluşturduğunda olaylar sırasını açıklayın. İç içe bağış nasıl işlenir?

lock_acquire() çağrıldığında öncelik bağışı şöyle oluyor:

1. Önce kilidin başka bir thread tarafından tutulup tutulmadığını kontrol ediyoruz.
2. Eğer tutuluyorsa, çalışan thread'in lock_waiting4 değişkenini bu kilide ayarlıyoruz.
3. Çalışan thread'in önceliği, kilidin max_priority değerinden yüksekse:
   - Kilidin max_priority değerini thread'in önceliğine yükseltiyoruz
   - thread_donate_priority() ile kilit sahibine öncelik bağışlıyoruz

İç içe bağış için:
4. Kilit sahibi de başka bir kilit için bekliyorsa (lock_waiting4 != NULL):
   - O kilidin max_priority değerini güncelliyoruz
   - O kilidin sahibine de öncelik bağışlıyoruz
   - Bu işlemi zincir boyunca tekrarlıyoruz

5. Sonra thread kilidi almak için semafor üzerinde bloke oluyor.
6. Kilidi aldığında:
   - lock_waiting4 değişkenini NULL yapıyoruz
   - Kilidi locks_holding listesine ekliyoruz
   - Kilidin sahibini kendisi olarak ayarlıyoruz

Bu şekilde öncelik bağışı zinciri oluşturuyoruz ve priority inversion problemini çözüyoruz.

>> B5: lock_release() fonksiyonu, yüksek öncelikli bir iş parçacığının beklediği bir kilit üzerinde çağrıldığında olaylar sırasını açıklayın.

lock_release() çağrıldığında:

1. thread_remove_lock() ile kilidi thread'in locks_holding listesinden çıkarıyoruz.
2. thread_update_priority() ile thread'in önceliğini güncelliyoruz:
   - Önceliği base_priority değerine geri döndürüyoruz
   - Eğer thread başka kilitler tutuyorsa ve bu kilitlerin max_priority değeri base_priority'den yüksekse, thread'in önceliğini en yüksek max_priority değerine ayarlıyoruz

3. Kilidin holder değişkenini NULL yapıyoruz.
4. sema_up() ile kilidin semaforunu serbest bırakıyoruz:
   - Bekleyen thread'ler arasından en yüksek öncelikli olanı seçiyoruz
   - Bu thread'i unblock ediyoruz
   - Eğer unblock edilen thread'in önceliği çalışan thread'den yüksekse, thread_yield() ile CPU'yu bırakıyoruz

Bu şekilde kilit serbest bırakıldığında, onu bekleyen en yüksek öncelikli thread hemen çalışmaya başlayabiliyor.

---- SENKRONİZASYON ----

>> B6: thread_set_priority() fonksiyonunda potansiyel bir yarış koşulunu açıklayın ve uygulamanızın bunu nasıl engellediğini açıklayın. Bu yarış koşulunu engellemek için bir kilit kullanabilir misiniz?

thread_set_priority() fonksiyonunda şöyle bir yarış koşulu olabilir:

1. Bir thread kendi önceliğini thread_set_priority() ile değiştirirken
2. Aynı anda başka bir thread ona öncelik bağışlamaya çalışırsa
3. Hangisinin son işlem olacağı belirsiz olur ve öncelik yanlış değerde kalabilir

Bunu engellemek için:
- thread_set_priority() başında intr_disable() ile kesmeleri kapatıyoruz
- thread_donate_priority() ve thread_update_priority() gibi öncelik değiştiren tüm fonksiyonlarda da kesmeleri kapatıyoruz
- İşlem bitince intr_set_level() ile kesmeleri tekrar açıyoruz

Bu şekilde öncelik değişiklikleri atomik oluyor ve yarış koşulları engelleniyor.

Kilit kullanmak yerine kesmeleri kapatmayı tercih ettik çünkü:
- Kilit kullanmak daha karmaşık olurdu
- Kilit için de öncelik bağışı gerekebilirdi, bu da recursive bir durum yaratabilirdi
- Kesmeleri kapatmak bu tür kısa işlemler için daha verimli
- Kilit kullanmak deadlock riski oluşturabilirdi

---- GEREKÇE ----

>> B7: Bu tasarımı neden seçtiniz? Diğer düşündüğünüz tasarımlara kıyasla hangi açılardan daha üstün olduğunu düşünüyorsunuz?

Bu tasarımı seçtik çünkü:

1. Hem doğrudan hem de iç içe öncelik bağışını destekliyor.
2. Veri yapıları basit ve anlaşılır.
3. Öncelik geri yükleme işlemi temiz bir şekilde yapılıyor.
4. Mevcut thread ve kilit yapılarına minimal değişiklikler yaparak çözüm ürettik.

Düşündüğümüz alternatif tasarım, her thread için ona öncelik bağışlayan thread'lerin bir listesini tutmaktı. Bu tasarımda:
- Her bağış ilişkisi için daha detaylı bilgi tutabilirdik
- Bağışların kaynağını daha net görebilirdik

Ama bu tasarımın dezavantajları vardı:
- Daha fazla bellek kullanımı gerektirecekti
- Liste işlemleri için daha fazla CPU zamanı harcanacaktı
- İç içe bağışları yönetmek daha karmaşık olacaktı
- Kod daha karmaşık ve hata yapmaya daha açık olacaktı

Sonuçta, seçtiğimiz tasarım daha basit, daha verimli ve daha az hata riski taşıyor. Ayrıca mevcut Pintos yapısına daha iyi entegre oluyor.

              GELİŞMİŞ PLANLAMA
              ==================

---- VERİ YAPILARI ----

>> C1: Her yeni veya değiştirilen `struct` veya `struct` üyesi, global veya statik değişken, `typedef` veya enumerasyonun beyanını buraya kopyalayın. Her birinin amacını 25 kelime veya daha az bir şekilde tanımlayın.

```c
/* thread.h içinde struct thread'e ekledik: */
int nice;                           /* Nice değeri */
fixed_t recent_cpu;                 /* Son CPU kullanımı */

/* thread.c içine ekledik: */
fixed_t load_avg;                   /* Sistem yük ortalaması */
```

nice: Thread'in nezaket değeri, -20 ile 20 arasında değişiyor. Yüksek değerler düşük öncelik anlamına geliyor.
recent_cpu: Thread'in son zamanlarda ne kadar CPU kullandığını gösteren değer. Sabit noktalı sayı olarak tutuyoruz.
load_avg: Sistemdeki hazır thread'lerin ortalama sayısı. Bu da sabit noktalı sayı.

Ayrıca fixed-point.h dosyasında sabit noktalı aritmetik için makrolar tanımladık.

---- ALGORİTMALAR ----

>> C2: A, B ve C iş parçacıklarının nice değerleri sırasıyla 0, 1 ve 2. Her birinin recent_cpu değeri 0. Aşağıdaki tabloyu doldurun ve her bir iş parçacığı için her bir zamanlayıcı tikinden sonra yapılan planlama kararını, öncelik ve recent_cpu değerlerini gösterin:

```
timer  recent_cpu    priority   thread  
ticks   A   B   C   A   B   C   to run  
-----  --  --  --  --  --  --   ------  
 0      0   0   0  63  61  59      A
 4      4   0   0  62  61  59      A
 8      8   0   0  61  61  59      A
12     12   0   0  60  61  59      B
16     12   4   0  60  60  59      A
20     16   4   0  59  60  59      B
24     16   8   0  59  59  59      A
28     20   8   0  58  59  59      B
32     20  12   0  58  58  59      C
36     20  12   4  58  58  58      A
```

Hesaplamalar:

0. tick:
   - Tüm thread'lerin recent_cpu = 0
   - Öncelik = PRI_MAX - (recent_cpu / 4) - (nice * 2)
   - A: 63 - (0 / 4) - (0 * 2) = 63
   - B: 63 - (0 / 4) - (1 * 2) = 61
   - C: 63 - (0 / 4) - (2 * 2) = 59
   - A en yüksek öncelikli, o çalışır

4. tick:
   - A 4 tick çalıştı, recent_cpu = 4
   - A: 63 - (4 / 4) - (0 * 2) = 62
   - B ve C değişmedi
   - A hala en yüksek öncelikli

8. tick:
   - A 4 tick daha çalıştı, recent_cpu = 8
   - A: 63 - (8 / 4) - (0 * 2) = 61
   - A ve B aynı öncelikte, ama A çalışmaya devam eder

12. tick:
   - A 4 tick daha çalıştı, recent_cpu = 12
   - A: 63 - (12 / 4) - (0 * 2) = 60
   - B şimdi daha yüksek öncelikli, B çalışır

16. tick:
   - B 4 tick çalıştı, recent_cpu = 4
   - B: 63 - (4 / 4) - (1 * 2) = 60
   - A ve B aynı öncelikte, sıra A'da

20. tick:
   - A 4 tick çalıştı, recent_cpu = 16
   - A: 63 - (16 / 4) - (0 * 2) = 59
   - B şimdi daha yüksek öncelikli, B çalışır

24. tick:
   - B 4 tick çalıştı, recent_cpu = 8
   - B: 63 - (8 / 4) - (1 * 2) = 59
   - A ve B aynı öncelikte, sıra A'da

28. tick:
   - A 4 tick çalıştı, recent_cpu = 20
   - A: 63 - (20 / 4) - (0 * 2) = 58
   - B şimdi daha yüksek öncelikli, B çalışır

32. tick:
   - B 4 tick çalıştı, recent_cpu = 12
   - B: 63 - (12 / 4) - (1 * 2) = 58
   - C şimdi en yüksek öncelikli, C çalışır

36. tick:
   - C 4 tick çalıştı, recent_cpu = 4
   - C: 63 - (4 / 4) - (2 * 2) = 58
   - Hepsi aynı öncelikte, sıra A'da

>> C3: Planlayıcı spesifikasyonundaki herhangi bir belirsizlik tablodaki değerleri belirsiz hale getirdi mi? Eğer öyleyse, bunları çözmek için hangi kuralı kullandınız? Bu, planlayıcınızın davranışıyla uyumlu mu?

Evet, birkaç belirsizlik vardı:

1. Eşit öncelikli thread'ler arasında hangisinin çalışacağı: Spesifikasyonda açıkça belirtilmemişti. Biz round-robin yaklaşımını kullandık, yani eşit öncelikli thread'ler sırayla çalışıyor.

2. Öncelik hesaplamasının ne zaman yapılacağı: Her 4 tick'te öncelikler güncelleniyor, ama bu güncellemenin planlama kararından önce mi sonra mı yapılacağı belirsizdi. Biz önce güncelleme, sonra planlama kararı yaklaşımını kullandık.

3. Çalışan thread'in ne zaman preempt edileceği: Başka bir thread'in önceliği çalışan thread'in önceliğine eşit olduğunda değil, sadece daha yüksek olduğunda preempt ediliyor.

Bizim planlayıcımız bu kurallara göre çalışıyor:
- Eşit öncelikli thread'ler arasında round-robin kullanıyoruz
- Öncelikleri her 4 tick'te ve planlama kararından önce güncelliyoruz
- Sadece daha yüksek öncelikli bir thread olduğunda preempt ediyoruz

Bu kurallar kodumuzla uyumlu ve mantıklı bir planlama stratejisi oluşturuyor.

>> C4: Planlamayı kod içinde ve kesme bağlamı dışında yapılan işlerin maliyeti arasında nasıl böldüğünüz, performansı nasıl etkiler?

Planlama işlerini şöyle böldük:

Kesme bağlamı içinde:
1. Her tick'te çalışan thread'in recent_cpu değerini artırma
2. Her saniye (TIMER_FREQ tick'te bir) load_avg ve tüm thread'lerin recent_cpu değerlerini güncelleme
3. Her 4 tick'te tüm thread'lerin önceliklerini güncelleme

Kesme bağlamı dışında:
1. Thread oluşturulduğunda öncelik hesaplama
2. Nice değeri değiştiğinde öncelik güncelleme
3. Thread'ler arası geçiş yapma

Bu bölünmenin performans etkileri:

Olumlu etkiler:
- En sık yapılan işlem (recent_cpu artırma) çok basit ve hızlı
- Ağır işlemler (tüm thread'leri güncelleme) daha seyrek yapılıyor
- Sabit noktalı aritmetik makroları hesaplamaları hızlandırıyor

Olumsuz etkiler:
- Her saniye ve her 4 tick'te yapılan toplu güncellemeler kesme süresini uzatıyor
- Thread sayısı arttıkça kesme süresi de artıyor

Performansı iyileştirmek için:
- Kayan nokta yerine sabit noktalı aritmetik kullandık
- Güncellemeleri her tick yerine belirli aralıklarla yapıyoruz
- Kesme işleyicisini olabildiğince basit tuttuk

Sonuçta bu denge, doğru planlama kararları ile sistem performansı arasında iyi bir uzlaşma sağlıyor.

---- GEREKÇE ----

>> C5: Tasarımınızı kısaca eleştirin, tasarım seçimlerinizdeki avantajları ve dezavantajları belirtin. Bu kısmı üzerinde daha fazla zaman harcayabilseydiniz, tasarımınızı nasıl geliştirmek veya iyileştirmek isterdiniz?

Tasarımımızın avantajları:
1. Sabit noktalı aritmetik için temiz bir soyutlama sağladık
2. BSD planlayıcı formüllerini doğru şekilde uyguladık
3. Mevcut thread yapısına minimal değişiklikler yaptık
4. Kayan nokta işlemleri yerine daha verimli sabit noktalı aritmetik kullandık

Dezavantajları:
1. Kesme işleyicisinde tüm thread'leri güncellemek kesme süresini uzatıyor
2. Sabit güncelleme aralıkları bazı iş yükleri için optimal olmayabilir
3. Thread sayısı arttıkça performans düşebilir

Daha fazla zamanımız olsaydı şunları yapardık:
1. Kesme işleyicisi dışında daha fazla iş yapacak şekilde tasarımı değiştirirdik
2. Tüm thread'leri bir kerede güncellemek yerine, her seferinde birkaç thread güncelleyerek yükü dağıtırdık
3. Sistem yüküne göre güncelleme sıklığını dinamik olarak ayarlayan bir mekanizma ekleyebilirdik
4. Öncelik değerlerini önbelleğe alıp, sadece gerektiğinde hesaplayabilirdik
5. Thread'leri önceliklerine göre daha verimli yönetmek için daha iyi veri yapıları kullanabilirdik

Aslında en çok zorlandığımız kısım sabit noktalı aritmetik oldu, çünkü kayan nokta işlemleri kullanamıyorduk ve integer aritmetiği ile hassas hesaplamalar yapmak zordu.

>> C6: Görev, sabit nokta matematiği için ayrıntılı aritmetiği açıklasa da, bunu nasıl uygulayacağınız konusunda size açık bir yön bırakmaktadır. Bunu yaptığınız şekilde uygulamaya karar vermenizin nedeni nedir? Sabit nokta matematiği için bir soyutlama katmanı, yani soyut veri tipi ve/veya sabit nokta sayıları manipüle etmek için bir dizi fonksiyon veya makro oluşturduysanız, bunu neden yaptınız? Yapmadıysanız, neden yapmadınız?

Sabit noktalı aritmetiği fixed-point.h dosyasında makrolar kullanarak uyguladık. Bunu seçmemizin nedenleri:

1. Verimlilik: Makrolar derleme zamanında genişletiliyor, bu da fonksiyon çağrısı ek yükünü ortadan kaldırıyor.
2. Okunabilirlik: FP_ADD, FP_MULT gibi anlamlı isimler kullanarak kodu daha anlaşılır hale getirdik.
3. Bakım kolaylığı: Tüm sabit noktalı işlemleri bir yerde topladık, değişiklik gerekirse sadece bu dosyayı güncellemek yeterli.
4. Tip güvenliği: fixed_t tipini tanımlayarak, hangi değişkenlerin sabit noktalı sayı olduğunu açıkça belirttik.

Makroları fonksiyonlar yerine seçtik çünkü:
- İşlemler basit ve hata kontrolü gerektirmiyor
- Planlayıcıda performans kritik öneme sahip
- Makrolar satır içi genişletilerek derleyici optimizasyonlarına daha fazla olanak sağlıyor

Kesirli kısım için 16 bit kullandık, bu da hem yeterli hassasiyet sağlıyor hem de geniş bir değer aralığına izin veriyor.

Başlangıçta fonksiyonlar kullanmayı düşündük, ama performans testlerinde makroların daha hızlı olduğunu gördük. Ayrıca makrolar kodu daha okunabilir hale getirdi, çünkü her yerde bit kaydırma ve maskeleme işlemleri yapmak yerine, anlamlı isimlerle işlem yapabiliyoruz.

Sonuçta, sabit noktalı aritmetik için soyutlama katmanı oluşturmak, hem kodu daha temiz ve anlaşılır hale getirdi, hem de performansı artırdı.
