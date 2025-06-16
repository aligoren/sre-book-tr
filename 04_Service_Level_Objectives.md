# Bölüm 4 - Service Level Objective'ler

**Yazan:** Chris Jones, John Wilkes ve Niall Murphy, Cody Smith ile birlikte  
**Editör:** Betsy Beyer

Bir servisi doğru şekilde yönetmek, hatta iyi yönetmek, o servis için gerçekten önemli olan davranışları anlamadan ve bu davranışları nasıl ölçüp değerlendireceğini bilmeden imkansızdır. Bu amaçla, kullanıcılarımıza belirli bir _servis seviyesi_ tanımlamak ve sunmak istiyoruz; ister dahili bir API kullansınlar ister halka açık bir ürün kullansınlar.

Kullanıcıların ne istediğine dair sezgi, deneyim ve anlayışımızı kullanarak _service level indicator'ları_ (SLI'lar), _objective'leri_ (SLO'lar) ve _agreement'ları_ (SLA'lar) tanımlıyoruz. Bu ölçümler önemli olan metriklerin temel özelliklerini, bu metriklerin hangi değerlere sahip olmasını istediğimizi ve beklenen servisi sağlayamazsak nasıl tepki vereceğimizi açıklar. Sonuçta, uygun metrikleri seçmek bir şeyler ters gittiğinde doğru eylemi yönlendirmeye yardımcı olur ve ayrıca bir SRE takımına servisin sağlıklı olduğu konusunda güven verir.

Bu bölüm metrik modelleme, metrik seçimi ve metrik analizi problemleriyle boğuşmak için kullandığımız framework'ü açıklar. Bu açıklamanın çoğu bir örnek olmadan oldukça soyut kalacağı için, ana noktalarımızı göstermek için Shakespeare: A Sample Service'te özetlenen Shakespeare servisini kullanacağız.

## Service Level Terminolojisi

Birçok okuyucu muhtemelen SLA konseptine aşinadır, ancak _SLI_ ve _SLO_ terimleri de dikkatli tanım gerektiriyor, çünkü yaygın kullanımda _SLA_ terimi aşırı yüklenmiş ve bağlama bağlı olarak bir dizi anlam kazanmıştır. Netlik için bu anlamları ayırmayı tercih ediyoruz.

## Indicator'lar

SLI bir service level _indicator_'ıdır—sağlanan servis seviyesinin bir yönünün dikkatli şekilde tanımlanmış nicel bir ölçüsü.

Çoğu servis _request latency_'yi—bir request'e yanıt döndürmenin ne kadar sürdüğünü—kilit bir SLI olarak görür. Diğer yaygın SLI'lar _error rate_'i (genellikle alınan tüm request'lerin bir oranı olarak ifade edilir) ve _system throughput_'u (tipik olarak saniye başına request olarak ölçülür) içerir. Ölçümler sıklıkla aggregate edilir: yani ham veri bir ölçüm penceresi boyunca toplanır ve sonra bir oran, ortalama veya percentile'a dönüştürülür.

İdeal olarak, SLI ilgilenilen bir servis seviyesini doğrudan ölçer, ancak bazen istenen ölçüm elde edilmesi veya yorumlanması zor olabileceği için sadece bir proxy mevcut olur. Örneğin, client-side latency genellikle kullanıcı için daha alakalı metrik olur, ancak sadece server'da latency ölçmek mümkün olabilir.

SRE'ler için önemli olan başka bir SLI türü _availability_'dir, yani bir servisin kullanılabilir olduğu zamanın oranı. Genellikle başarılı olan iyi biçimlendirilmiş request'lerin oranı açısından tanımlanır, bazen _yield_ olarak adlandırılır. (_Durability_—verinin uzun bir süre boyunca korunma olasılığı—veri depolama sistemleri için eşit derecede önemlidir.) %100 availability imkansız olmasına rağmen, %100'e yakın availability genellikle kolayca elde edilebilir ve endüstri yaygın olarak yüksek availability değerlerini availability yüzdesindeki "nine" sayısı açısından ifade eder. Örneğin, %99 ve %99.999 availability'ler sırasıyla "2 nine" ve "5 nine" availability olarak adlandırılabilir ve Google Compute Engine availability için mevcut yayınlanan target "üç buçuk nine"—%99.95 availability'dir.

## Objective'ler

SLO bir _service level objective_'idir: bir SLI tarafından ölçülen bir servis seviyesi için target değer veya değer aralığı. SLO'lar için doğal yapı böylece _SLI ≤ target_ veya _lower bound ≤ SLI ≤ upper bound_'dur. Örneğin, Shakespeare arama sonuçlarını "hızlı" döndürmeye karar verebilir ve ortalama arama request latency'mizin 100 milisaniyeden az olması gerektiği şeklinde bir SLO benimseriz.

Uygun bir SLO seçmek karmaşıktır. Başlangıç olarak, her zaman değerini seçme şansınız yoktur! Dış dünyadan servisinize gelen HTTP request'ler için, queries per second (QPS) metriği esasen kullanıcılarınızın istekleri tarafından belirlenir ve bunun için gerçekten bir SLO belirleyemezsiniz.

Öte yandan, request başına ortalama latency'nin 100 milisaniyenin altında olmasını istediğinizi _söyleyebilirsiniz_ ve böyle bir hedef belirlemek sizi frontend'inizi çeşitli türlerde düşük latency davranışlarıyla yazmaya veya belirli türlerde düşük latency ekipmanları satın almaya motive edebilir. (100 milisaniye açıkça keyfi bir değerdir, ancak genel olarak düşük latency sayıları iyidir. Hızlının yavaştan daha iyi olduğuna ve belirli değerlerin üzerindeki kullanıcı deneyimli latency'nin aslında insanları uzaklaştırdığına inanmak için mükemmel nedenler vardır—daha fazla detay için "Speed Matters" [Bru09]'a bakın.)

Yine, bu ilk göründüğünden daha inceliklidir, çünkü bu iki SLI—QPS ve latency—perde arkasında bağlantılı olabilir: daha yüksek QPS genellikle daha büyük latency'lere yol açar ve servislerin bazı yük eşiğinin ötesinde bir performans uçurumuna sahip olması yaygındır.

SLO'ları seçmek ve kullanıcılara yayınlamak bir servisin nasıl performans göstereceği konusunda beklentiler belirler. Bu strateji, örneğin servisin yavaş olması hakkında servis sahiplerine yönelik asılsız şikayetleri azaltabilir. Açık bir SLO olmadan, kullanıcılar genellikle istenen performans hakkında kendi inançlarını geliştirirler, bu da servisi tasarlayan ve işleten kişilerin sahip olduğu inançlarla ilgisiz olabilir. Bu dinamik hem servise aşırı bağımlılığa (kullanıcılar yanlış şekilde bir servisin gerçekte olduğundan daha fazla available olacağına inandıklarında, Chubby'de olduğu gibi: The Global Chubby Planned Outage'a bakın) hem de yetersiz bağımlılığa (potansiyel kullanıcılar bir sistemin gerçekte olduğundan daha kararsız ve daha az güvenilir olduğuna inandıklarında) yol açabilir.

##### The Global Chubby Planned Outage

**Yazan:** Marc Alvidrez

Chubby [Bur06] Google'ın gevşek bağlı dağıtık sistemler için lock servisidir. Global durumda, Chubby instance'larını her replica'nın farklı bir coğrafi bölgede olacak şekilde dağıtırız. Zamanla, global Chubby instance'ının arızalarının tutarlı şekilde servis outage'ları oluşturduğunu, bunların çoğunun son kullanıcılara görünür olduğunu bulduk. Ortaya çıktığına göre, gerçek global Chubby outage'ları o kadar nadir ki servis sahipleri Chubby'nin asla düşmeyeceğini varsayarak ona bağımlılıklar eklemeye başladılar. Yüksek güvenilirliği yanlış bir güvenlik hissi sağladı çünkü servisler Chubby kullanılamadığında uygun şekilde çalışamıyordu, ne kadar nadir olursa olsun.

Bu Chubby senaryosunun çözümü ilginçtir: SRE global Chubby'nin service level objective'ini karşıladığından, ancak önemli ölçüde aşmadığından emin olur. Herhangi bir çeyrekte, gerçek bir arıza availability'yi target'ın altına düşürmediyse, sistemi kasıtlı olarak kapatarak kontrollü bir outage sentezlenir. Bu şekilde, Chubby'ye makul olmayan bağımlılıkları eklendikten kısa süre sonra ortaya çıkarabiliriz. Bunu yapmak servis sahiplerini dağıtık sistemlerin gerçekliğiyle daha geç değil daha erken yüzleşmeye zorlar.

## Agreement'lar

Son olarak, SLA'lar service level _agreement'larıdır: kullanıcılarınızla içerdikleri SLO'ları karşılama (veya kaçırma) sonuçlarını içeren açık veya örtük sözleşme. Sonuçlar finansal olduklarında—bir indirim veya ceza—en kolay tanınırlar, ancak başka biçimler de alabilirler. SLO ve SLA arasındaki farkı söylemenin kolay bir yolu "SLO'lar karşılanmazsa ne olur?" diye sormaktır: açık bir sonuç yoksa, neredeyse kesinlikle bir SLO'ya bakıyorsunuz.

SRE tipik olarak SLA'ları oluşturmaya dahil olmaz, çünkü SLA'lar business ve ürün kararlarıyla yakından bağlantılıdır. Ancak SRE, kaçırılan SLO'ların sonuçlarını tetiklemeyi önlemeye yardımcı olmaya dahil olur. Ayrıca SLI'ları tanımlamaya yardımcı olabilirler: agreement'taki SLO'ları ölçmek için açıkça objektif bir yol olması gerekir, yoksa anlaşmazlıklar çıkar.

Google Search, halka açık bir SLA'sı olmayan önemli bir servis örneğidir: herkesin Search'ü mümkün olduğunca akıcı ve verimli kullanmasını istiyoruz, ancak tüm dünyayla bir sözleşme imzalamadık. Yine de, Search kullanılamadığında hala sonuçlar vardır—kullanılamazlık itibarımıza bir darbe ve reklam gelirinde bir düşüşle sonuçlanır. Google for Work gibi diğer birçok Google servisi, kullanıcılarıyla açık SLA'lara sahiptir. Belirli bir servisin SLA'sı olsun ya da olmasın, SLI'ları ve SLO'ları tanımlamak ve bunları servisi yönetmek için kullanmak değerlidir.

Teori bu kadar—şimdi deneyime geçelim.

# Indicator'lar Pratikte

Servisinizi ölçmek için uygun metrikleri seçmenin _neden_ önemli olduğu konusunda argümanımızı yaptığımıza göre, servisiniz veya sisteminiz için anlamlı olan metrikleri nasıl belirleyeceğiniz konusuna geçelim?

## Siz ve Kullanıcılarınız Neyi Önemsiyor?

Monitoring sisteminizde takip edebileceğiniz her metriği SLI olarak kullanmamalısınız; kullanıcılarınızın sistemden ne istediğini anlamak, birkaç indicator'ın akıllıca seçimini bilgilendirecektir. Çok fazla indicator seçmek önemli olan indicator'lara doğru seviyede dikkat etmeyi zorlaştırır, çok az seçmek ise sisteminizin önemli davranışlarını incelenmemiş bırakabilir. Tipik olarak bir avuç temsili indicator'ın bir sistemin sağlığını değerlendirmek ve hakkında akıl yürütmek için yeterli olduğunu buluyoruz.

Servisler, alakalı buldukları SLI'lar açısından birkaç geniş kategoriye ayrılma eğilimindedir:

* Shakespeare arama frontend'leri gibi _User-facing serving sistemler_, genellikle _availability_, _latency_ ve _throughput_'u önemser. Başka bir deyişle: Request'e yanıt verebildik mi? Yanıt vermek ne kadar sürdü? Kaç request işlenebilir?
* _Storage sistemler_ genellikle _latency_, _availability_ ve _durability_'yi vurgular. Başka bir deyişle: Veri okumak veya yazmak ne kadar sürüyor? Talep üzerine veriye erişebiliyor muyuz? İhtiyacımız olduğunda veri hala orada mı? Bu konuların genişletilmiş tartışması için Data Integrity: What You Read Is What You Wrote'a bakın.
* Veri işleme pipeline'ları gibi _Big data sistemler_, _throughput_ ve _end-to-end latency_'yi önemseme eğilimindedir. Başka bir deyişle: Ne kadar veri işleniyor? Verinin ingestion'dan completion'a ilerlemesi ne kadar sürüyor? (Bazı pipeline'lar ayrıca bireysel işleme aşamaları için latency target'larına sahip olabilir.)
* Tüm sistemler _correctness_'ı önemsemelidir: doğru cevap döndürüldü mü, doğru veri alındı mı, doğru analiz yapıldı mı? Correctness, genellikle infrastructure'ın _per se_ bir özelliği değil sistemdeki verinin bir özelliği olmasına ve dolayısıyla genellikle karşılanması SRE sorumluluğu olmamasına rağmen, sistem sağlığının bir indicator'ı olarak takip edilmesi önemlidir.

## Indicator'ları Toplama

Birçok indicator metriği en doğal olarak server tarafında, Borgmon (Practical Alerting from Time-Series Data'ya bakın) veya Prometheus gibi bir monitoring sistemi kullanarak veya periyodik log analizi ile toplanır—örneğin, tüm request'lerin bir oranı olarak HTTP 500 yanıtları. Ancak, bazı sistemler _client_-side toplama ile enstrümante edilmelidir, çünkü client'ta davranışı ölçmemek kullanıcıları etkileyen ancak server-side metrikleri etkilemeyen bir dizi problemi kaçırabilir. Örneğin, Shakespeare arama backend'inin yanıt latency'sine odaklanmak, sayfanın JavaScript'i ile ilgili problemler nedeniyle kötü kullanıcı latency'sini kaçırabilir: bu durumda, bir sayfanın browser'da kullanılabilir hale gelmesinin ne kadar sürdüğünü ölçmek kullanıcının gerçekte deneyimlediği şey için daha iyi bir proxy'dir.

## Aggregation

Basitlik ve kullanılabilirlik için, ham ölçümleri sıklıkla aggregate ederiz. Bu dikkatli yapılması gerekir.

Bazı metrikler _saniye başına_ servis edilen request sayısı gibi görünüşte basittir, ancak bu görünüşte basit ölçüm bile ölçüm penceresi boyunca veriyi örtük olarak aggregate eder. Ölçüm saniyede bir mi alınıyor, yoksa bir dakika boyunca request'leri ortalayarak mı? İkincisi sadece birkaç saniye süren burst'larda çok daha yüksek anlık request oranlarını gizleyebilir. Çift numaralı saniyelerde 200 request/s servis eden ve diğerlerinde 0 servis eden bir sistemi düşünün. Sabit 100 request/s servis eden bir sistemle aynı ortalama yüke sahiptir, ancak _ortalama_ olandan iki kat büyük _anlık_ yüke sahiptir. Benzer şekilde, request latency'lerini ortalamak çekici görünebilir, ancak önemli bir detayı gizler: request'lerin çoğunun hızlı olması, ancak uzun bir request kuyruğunun çok, çok daha yavaş olması tamamen mümkündür.

Çoğu metrik ortalamalardan ziyade _dağılımlar_ olarak düşünülmesi daha iyidir. Örneğin, bir latency SLI için, bazı request'ler hızla servis edilirken, diğerleri kaçınılmaz olarak daha uzun sürer—bazen çok daha uzun. Basit bir ortalama bu tail latency'leri ve bunlardaki değişiklikleri gizleyebilir. Şekil 4-1 bir örnek sağlar: tipik bir request yaklaşık 50 ms'de servis edilmesine rağmen, request'lerin %5'i 20 kat daha yavaştır! Sadece ortalama latency'ye dayalı monitoring ve alerting, gün boyunca davranışta hiçbir değişiklik göstermezken, aslında tail latency'de önemli değişiklikler vardır (en üstteki çizgi).

![Şekil 4-1. Bir sistem için 50., 85., 95. ve 99. percentile latency'ler. Y-ekseninin logaritmik ölçeği olduğuna dikkat edin.](figure-4-1.png)

Indicator'lar için percentile'ları kullanmak dağılımın şeklini ve farklı özelliklerini düşünmenizi sağlar: 99. veya 99.9. gibi yüksek dereceli bir percentile size makul bir worst-case değeri gösterirken, 50. percentile'ı (median olarak da bilinir) kullanmak tipik durumu vurgular. Yanıt sürelerindeki varyans ne kadar yüksekse, tipik kullanıcı deneyimi o kadar çok long-tail davranışından etkilenir, bu etki yüksek yükte queuing etkileri tarafından daha da kötüleşir. Kullanıcı çalışmaları insanların tipik olarak yanıt süresinde yüksek varyansa sahip olandan biraz daha yavaş bir sistemi tercih ettiklerini göstermiştir, bu yüzden bazı SRE takımları sadece yüksek percentile değerlerine odaklanır, 99.9. percentile davranışı iyi ise tipik deneyimin kesinlikle iyi olacağı gerekçesiyle.

##### İstatistiksel Yanılgılar Üzerine Bir Not

Genellikle bir değer setinin ortalaması (aritmetik ortalama) yerine percentile'larla çalışmayı tercih ederiz. Bunu yapmak, genellikle ortalamadan önemli ölçüde farklı (ve daha ilginç) özelliklere sahip olan uzun veri noktası kuyruğunu düşünmeyi mümkün kılar. Bilgisayar sistemlerinin yapay doğası nedeniyle, veri noktaları genellikle çarpıktır—örneğin, hiçbir request 0 ms'den az yanıta sahip olamaz ve 1.000 ms'de bir timeout, timeout'tan büyük değerlere sahip başarılı yanıtların olamayacağı anlamına gelir. Sonuç olarak, ortalama ve median'ın aynı—hatta birbirine yakın—olduğunu varsayamayız!

Önce doğrulamadan verilerimizin normal dağıldığını varsaymamaya çalışırız, bazı standart sezgiler ve yaklaşımların geçerli olmaması durumunda. Örneğin, dağılım beklenenden farklıysa, outlier'ları gördüğünde eylem alan bir süreç (örneğin, yüksek request latency'leri olan bir server'ı yeniden başlatmak) bunu çok sık veya yeterince sık yapmayabilir.

## Indicator'ları Standardize Etme

Her seferinde ilk prensiplerden akıl yürütmek zorunda kalmamak için SLI'lar için ortak tanımlarda standardize etmenizi öneririz. Standart tanım şablonlarına uyan herhangi bir özellik, bireysel bir SLI'ın spesifikasyonundan çıkarılabilir, örneğin:

* Aggregation aralıkları: "1 dakika boyunca ortalaması alınmış"
* Aggregation bölgeleri: "Bir cluster'daki tüm task'lar"
* Ölçümlerin ne sıklıkla yapıldığı: "Her 10 saniyede bir"
* Hangi request'lerin dahil edildiği: "Black-box monitoring job'larından HTTP GET'ler"
* Verinin nasıl elde edildiği: "Monitoring'imiz aracılığıyla, server'da ölçülmüş"
* Data-access latency: "Son byte'a kadar geçen süre"

Çabadan tasarruf etmek için, her yaygın metrik için yeniden kullanılabilir SLI şablonları seti oluşturun; bunlar ayrıca herkesin belirli bir SLI'ın ne anlama geldiğini anlamasını basitleştirir.

# Objective'ler Pratikte

Kullanıcılarınızın neyi önemsediğini düşünerek (veya öğrenerek!) başlayın, neyi ölçebileceğinizi değil. Genellikle, kullanıcılarınızın önemsediği şey ölçülmesi zor veya imkansızdır, bu yüzden kullanıcıların ihtiyaçlarını bir şekilde yaklaşık olarak karşılamak zorunda kalacaksınız. Ancak, sadece ölçmesi kolay olanla başlarsanız, daha az faydalı SLO'larla sonuçlanırsınız. Sonuç olarak, bazen istenen objective'lerden geriye doğru belirli indicator'lara çalışmanın indicator'ları seçip sonra target'lar bulmaktan daha iyi çalıştığını bulduk.

## Objective'leri Tanımlama

Maksimum netlik için, SLO'lar nasıl ölçüldüklerini ve hangi koşullar altında geçerli olduklarını belirtmelidir. Örneğin, şunları söyleyebiliriz (ikinci satır birincisiyle aynıdır, ancak redundancy'yi kaldırmak için önceki bölümün SLI varsayılanlarına dayanır):

* `Get` RPC çağrılarının %99'u (1 dakika boyunca ortalaması alınmış) 100 ms'den az sürede tamamlanacak (tüm backend server'larda ölçülmüş).
* `Get` RPC çağrılarının %99'u 100 ms'den az sürede tamamlanacak.

Performans eğrilerinin şekli önemliyse, birden fazla SLO target'ı belirtebilirsiniz:

* `Get` RPC çağrılarının %90'ı 1 ms'den az sürede tamamlanacak.
* `Get` RPC çağrılarının %99'u 10 ms'den az sürede tamamlanacak.
* `Get` RPC çağrılarının %99.9'u 100 ms'den az sürede tamamlanacak.

Throughput'u önemseyen bir bulk processing pipeline ve latency'yi önemseyen bir interactive client gibi heterojen workload'lara sahip kullanıcılarınız varsa, her workload sınıfı için ayrı objective'ler tanımlamak uygun olabilir:

* Throughput client'larının `Set` RPC çağrılarının %95'i < 1 s'de tamamlanacak.
* < 1 kB payload'lara sahip latency client'larının `Set` RPC çağrılarının %99'u < 10 ms'de tamamlanacak.

SLO'ların %100 zamanında karşılanacağında ısrar etmek hem gerçekçi değil hem de istenmeyen bir durumdur: bunu yapmak inovasyon ve deployment oranını azaltabilir, pahalı, aşırı muhafazakar çözümler gerektirebilir veya her ikisini birden yapabilir. Bunun yerine, bir error budget'ına—SLO'ların kaçırılabileceği bir orana—izin vermek ve bunu günlük veya haftalık bazda takip etmek daha iyidir. Üst yönetim muhtemelen aylık veya çeyreklik bir değerlendirme de isteyecektir. (Error budget, diğer SLO'ları karşılamak için sadece bir SLO'dur!)

SLO ihlal oranı error budget ile karşılaştırılabilir (Motivation for Error Budgets'a bakın), aradaki fark yeni release'leri ne zaman çıkaracağına karar veren sürecin bir girdisi olarak kullanılır.

## Target'ları Seçme

Target'ları (SLO'ları) seçmek tamamen teknik bir aktivite değildir çünkü seçilen SLI'lara ve SLO'lara (ve belki SLA'lara) yansıtılması gereken ürün ve business etkileri vardır. Benzer şekilde, personel, pazara çıkış süresi, donanım kullanılabilirliği ve finansman tarafından oluşturulan kısıtlamalar içinde belirli ürün özelliklerini diğerlerine karşı takas etmek gerekli olabilir. SRE bu konuşmanın bir parçası olmalı ve farklı seçeneklerin riskleri ve uygulanabilirliği konusunda tavsiyelerde bulunmalı olsa da, bu tartışmayı daha verimli hale getirmeye yardımcı olabilecek birkaç ders öğrendik:

**Mevcut performansa dayalı bir target seçmeyin**

Bir sistemin değerlerini ve sınırlarını anlamak esassel olsa da, değerleri düşünce olmadan benimsemek sizi target'larını karşılamak için kahramanca çabalar gerektiren ve önemli yeniden tasarım olmadan geliştirilemeyecek bir sistemi desteklemeye kilitleyebilir.

**Basit tutun**

SLI'lardaki karmaşık aggregation'lar sistem performansındaki değişiklikleri gizleyebilir ve ayrıca hakkında akıl yürütmesi daha zordur.

**Mutlak ifadelerden kaçının**

Yükünü "sonsuz" şekilde ölçekleyebilen ve "her zaman" kullanılabilir olan bir sistem istemek cazip olsa da, bu gereksinim gerçekçi değildir. Böyle ideallere yaklaşan bir sistem bile muhtemelen tasarlanması ve inşa edilmesi uzun zaman alacak ve işletilmesi pahalı olacaktır—ve muhtemelen kullanıcıların mutlu (hatta memnun) olmak için sahip olmaktan memnun olacaklarından gereksiz yere daha iyi olacaktır.

**Mümkün olduğunca az SLO'ya sahip olun**

Sisteminizin özelliklerinin iyi kapsamını sağlamak için yeterli SLO seçin. Seçtiğiniz SLO'ları savunun: belirli bir SLO'yu alıntılayarak öncelikler hakkında bir konuşmayı asla kazanamazsanız, muhtemelen o SLO'ya sahip olmaya değmez. Ancak, tüm ürün özellikleri SLO'lara uygun değildir: "kullanıcı memnuniyeti"ni bir SLO ile belirtmek zordur.

**Mükemmellik bekleyebilir**

SLO tanımlarını ve target'larını bir sistem davranışı hakkında öğrendikçe zaman içinde her zaman iyileştirebilirsiniz. Ulaşılamaz olduğunu keşfettiğinizde gevşetilmesi gereken aşırı katı bir target seçmekten ziyade sıkılaştırdığınız gevşek bir target ile başlamak daha iyidir.

SLO'lar SRE'ler ve ürün geliştiricileri için işi önceliklendirmede büyük bir itici güç olabilir ve olmalıdır, çünkü kullanıcıların önemsediği şeyi yansıtırlar. İyi bir SLO, bir geliştirme takımı için yardımcı, meşru bir zorlayıcı fonksiyondur. Ancak kötü düşünülmüş bir SLO, bir takım aşırı agresif bir SLO'yu karşılamak için kahramanca çabalar kullanırsa boşa giden işle veya SLO çok gevşekse kötü bir ürünle sonuçlanabilir. SLO'lar büyük bir kaldıraçtır: onları akıllıca kullanın.

## Kontrol Önlemleri

SLI'lar ve SLO'lar sistemleri yönetmek için kullanılan kontrol döngülerinin kritik öğeleridir:

1. Sistemin SLI'larını monitör edin ve ölçün.
2. SLI'ları SLO'larla karşılaştırın ve eylem gerekip gerekmediğine karar verin.
3. Eylem gerekiyorsa, target'ı karşılamak için _ne_ olması gerektiğini bulun.
4. O eylemi gerçekleştirin.

Örneğin, adım 2 request latency'nin arttığını ve bir şey yapılmadığı takdirde birkaç saat içinde SLO'yu kaçıracağını gösteriyorsa, adım 3 server'ların CPU-bound olduğu hipotezini test etmeyi ve yükü yaymak için daha fazlasını eklemeye karar vermeyi içerebilir. SLO olmadan, ne zaman (veya ne zaman) eylem alacağınızı bilemezsiniz.

## SLO'lar Beklentileri Belirler

SLO'ları yayınlamak sistem davranışı için beklentiler belirler. Kullanıcılar (ve potansiyel kullanıcılar) genellikle bir servisin kullanım durumları için uygun olup olmadığını anlamak için ondan ne bekleyebileceklerini bilmek isterler. Örneğin, bir fotoğraf paylaşım websitesi oluşturmak isteyen bir takım, çok güçlü durability ve düşük maliyet vaat eden ancak biraz daha düşük availability karşılığında bir servis kullanmaktan kaçınmak isteyebilir, aynı servis bir arşiv kayıt yönetim sistemi için mükemmel bir uyum olabilir.

Kullanıcılarınız için gerçekçi beklentiler belirlemek için, aşağıdaki taktiklerden birini veya her ikisini kullanmayı düşünebilirsiniz:

**Güvenlik marjı tutun**

Kullanıcılara reklamı yapılan SLO'dan daha sıkı bir dahili SLO kullanmak, kronik problemlere dışarıdan görünür hale gelmeden önce yanıt vermeniz için alan sağlar. Bir SLO buffer'ı ayrıca performansı maliyet veya bakım kolaylığı gibi diğer özellikler için takas eden yeniden implementasyonları kullanıcıları hayal kırıklığına uğratmak zorunda kalmadan barındırmayı mümkün kılar.

**Aşırı başarı göstermeyin**

Kullanıcılar, özellikle infrastructure servisleri için, söylediğiniz şeyi sağlayacağınızdan ziyade gerçekte sunduğunuz şeyin üzerine inşa ederler. Servisinizin gerçek performansı belirtilen SLO'sundan çok daha iyiyse, kullanıcılar mevcut performansına bağımlı hale geleceklerdir. Sistemi kasıtlı olarak ara sıra offline alarak (Google'ın Chubby servisi aşırı kullanılabilir olmaya yanıt olarak planlı outage'lar getirdi), bazı request'leri throttle ederek veya sistemi hafif yükler altında daha hızlı olmayacak şekilde tasarlayarak aşırı bağımlılığı önleyebilirsiniz.

Bir sistemin beklentilerini ne kadar iyi karşıladığını anlamak, sistemi daha hızlı, daha kullanılabilir ve daha dayanıklı hale getirmek için yatırım yapıp yapmayacağına karar vermeye yardımcı olur. Alternatif olarak, servis iyi gidiyorsa, belki personel zamanı teknik borcu ödeme, yeni özellikler ekleme veya diğer ürünleri tanıtma gibi diğer önceliklere harcanmalıdır.

# Agreement'lar Pratikte

Bir SLA oluşturmak, business ve hukuk takımlarının bir ihlal için uygun sonuçları ve cezaları seçmesini gerektirir. SRE'nin rolü, SLA'da yer alan SLO'ları karşılamanın olasılığını ve zorluğunu anlamalarına yardımcı olmaktır. SLO yapısı hakkındaki tavsiyelerin çoğu SLA'lar için de geçerlidir. Kullanıcılara reklamını yaptığınız şeyde muhafazakar olmak akıllıcadır, çünkü seçmen kitlesi ne kadar geniş olursa, akıllıca olmayan veya çalışması zor olan SLA'ları değiştirmek veya silmek o kadar zor olur. 