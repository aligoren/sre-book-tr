# Bölüm 6 - Distributed System'leri İzlemek

*Yazan: Rob Ewaschuk*  
*Editör: Betsy Beyer*

Google'ın SRE ekiplerinin başarılı izleme ve uyarı sistemleri oluşturmak için bazı temel prensipleri ve en iyi uygulamaları vardır. Bu bölüm, hangi sorunların bir çağrı (page) aracılığıyla insanı rahatsız etmesi gerektiği konusunda rehberlik sunar ve çağrı tetiklemeye yetecek kadar ciddi olmayan sorunlarla nasıl başa çıkılacağını ele alır.

## Tanımlar 

İzleme ile ilgili tüm konuları tartışmak için evrensel olarak paylaşılan bir kelime dağarcığı yoktur. Google içinde bile bu terimlerin kullanımı değişiklik gösterir, ancak en yaygın yorumlar burada listelenmiştir.

**İzleme (Monitoring)**

Bir sistem hakkında sorgu sayıları ve türleri, hata sayıları ve türleri, işlem süreleri ve sunucu yaşam süreleri gibi gerçek zamanlı nicel verileri toplama, işleme, birleştirme ve görüntüleme.

**White-box izleme**

Sistemin iç yapısından çıkarılan metrikler üzerine dayalı izleme, log'lar, Java Virtual Machine Profiling Interface gibi interface'ler veya dahili istatistikler yayan HTTP handler'lar dahil.

**Black-box izleme**

Bir kullanıcının göreceği şekilde dışarıdan görünen davranışları test etme.

**Kontrol paneli (Dashboard)**

Bir servisin temel metriklerinin özet görünümünü sağlayan uygulama (genellikle web tabanlı). Bir kontrol paneli filtreleri, seçicileri vs. içerebilir, ancak kullanıcıları için en önemli metrikleri açığa çıkaracak şekilde önceden oluşturulmuştur. Kontrol paneli ayrıca ticket kuyruğu uzunluğu, yüksek öncelikli hataların listesi, belirli bir sorumluluk alanı için mevcut nöbetçi mühendisi veya son güncellemeler gibi ekip bilgilerini de gösterebilir.

**Uyarı (Alert)**

Bir insan tarafından okunması amaçlanan ve hata veya ticket kuyruğu, e-posta listesi veya çağrı cihazı gibi bir sisteme gönderilen bildirim. Sırasıyla bu uyarılar *ticket'lar*, *e-posta uyarıları*<sup>22</sup> ve *çağrılar* olarak sınıflandırılır.

**Root cause**

Eğer onarılırsa, bu olayın aynı şekilde tekrar olmayacağına dair güven veren bir yazılım veya insan sistemindeki kusur. Belirli bir incident'ın birden fazla root cause'u olabilir: örneğin, yetersiz process automation, hatalı input'ta çöken yazılım *ve* configuration oluşturmak için kullanılan script'in yetersiz test edilmesi kombinasyonundan kaynaklanmış olabilir. Bu faktörlerin her biri tek başına root cause olarak durabilir ve her biri onarılmalıdır.

**Node ve machine**

Fiziksel sunucu, sanal makine veya container'da çalışan bir kernel'in tek bir instance'ını belirtmek için birbirinin yerine kullanılır. Tek bir machine'de monitor edilmeye değer birden fazla *servis* olabilir. Servisler şu şekillerde olabilir:

- Birbirleriyle ilişkili: örneğin, caching server ve web server
- Hardware paylaşan ilişkisiz servisler: örneğin, kod repository'si ve Puppet veya Chef gibi configuration sistemi için master

**Push**

Servisin çalışan yazılımına veya configuration'una yapılan herhangi bir değişiklik.

## Neden İzleme Yaparız?

Bir sistemi izlemek için birçok neden vardır:

**Uzun vadeli trendleri analiz etmek**

Veritabanım ne kadar büyük ve ne kadar hızlı büyüyor? Günlük aktif kullanıcı sayım ne kadar hızlı artıyor?

**Zaman içinde veya deney grupları üzerinde karşılaştırma yapmak**

Sorgular Acme Bucket of Bytes 2.72 ile Ajax DB 3.14'e göre daha mı hızlı? Ekstra bir düğüm ile önbellek isabet oranım ne kadar daha iyi? Sitem geçen haftakine göre daha mı yavaş?

**Uyarı vermek**

Bir şey bozuk ve birinin hemen düzeltmesi gerekiyor! Veya bir şey yakında bozulabilir, o yüzden birinin yakında bakması gerekiyor.

**Kontrol panelleri oluşturmak**

Kontrol panelleri servisiniz hakkında temel soruları cevaplamalı ve normalde dört altın sinyalin bir biçimini içermelidir.

**Geçmişe dönük analiz yapmak (yani, hata ayıklama)**

Gecikmemiz aniden arttı; aynı zamanda başka ne oldu?

Sistem izleme ayrıca iş analitiğine ham veri sağlamada ve güvenlik ihlallerinin analizini kolaylaştırmada yardımcı olur. Bu kitap SRE'nin özel uzmanlığa sahip olduğu mühendislik alanlarına odaklandığı için, burada izlemenin bu uygulamalarını tartışmayacağız.

İzleme ve uyarı, bir sistemin ne zaman bozuk olduğunu veya belki de neyin bozulmak üzere olduğunu bize söylemesini sağlar. Sistem kendini otomatik olarak düzeltemediğinde, bir insanın uyarıyı araştırmasını, elinde gerçek bir problem olup olmadığını belirlemesini, problemi hafifletmesini ve problemin temel nedenini belirlemesini isteriz. Çok dar kapsamlı sistem bileşenleri üzerinde güvenlik denetimi yapmadığınız sürece, sadece "bir şey biraz garip görünüyor" diye asla uyarı tetiklememalisiniz.

Bir insanı çağırmak çalışanın zamanının oldukça pahalı bir kullanımıdır. Eğer çalışan işte ise, çağrı iş akışını böler. Eğer çalışan evde ise, çağrı kişisel zamanını ve belki de uykusunu böler. Çağrılar çok sık gerçekleştiğinde, çalışanlar gelen uyarıları ikinci kez tahmin eder, gözden geçirir veya hatta yok sayar, bazen gürültü tarafından maskelenen "gerçek" bir çağrıyı bile yok sayar. Diğer gürültüler hızlı teşhis ve düzeltmeyi engellediği için kesintiler uzayabilir. Etkili uyarı sistemlerinin iyi sinyali ve çok düşük gürültüsü vardır.

## İzleme İçin Makul Beklentiler Belirlemek

Karmaşık bir uygulamayı izlemek kendi başına önemli bir mühendislik çabasıdır. Enstrümantasyon, veri toplama, görüntüleme ve uyarı için mevcut önemli altyapı bile olsa, 10-12 üyeli bir Google SRE ekibinde genellikle bir ve bazen iki üye vardır ki onların birincil görevi servisleri için izleme sistemlerini oluşturmak ve sürdürmektir. Bu sayı, yaygın izleme altyapısını genelleştirip merkezileştirdikçe zamanla azalmıştır, ancak her SRE ekibinde genellikle en az bir "izleme uzmanı" vardır.

Genel olarak Google, sonradan analiz için daha iyi araçlarla birlikte daha basit ve daha hızlı izleme sistemlerine yönelmiştir. Eşikleri öğrenmeye veya nedenselliği otomatik olarak algılamaya çalışan "sihirli" sistemlerden kaçınırız. Son kullanıcı istek oranlarındaki beklenmedik değişiklikleri algılayan kurallar bir karşı örnektir; bu kurallar mümkün olduğunca basit tutulsa da, çok basit, spesifik, ciddi bir anormalliğin çok hızlı algılanmasını sağlar. Kapasite planlama ve trafik tahmini gibi izleme verisinin diğer kullanımları daha fazla kırılganlığı, dolayısıyla daha fazla karmaşıklığı tolere edebilir. Çok uzun zaman ufkunda (aylar veya yıllar) düşük örnekleme oranıyla (saatler veya günler) yürütülen gözlemsel deneyler de genellikle daha fazla kırılganlığı tolere edebilir çünkü ara sıra kaçırılan örnekler uzun süreli eğilimi gizlemez.

Google SRE karmaşık bağımlılık hiyerarşileri ile sadece sınırlı başarı yaşamıştır. "Eğer veritabanının yavaş olduğunu biliyorsam, yavaş veritabanı için alert et; aksi takdirde, web sitesinin genel olarak yavaş olması için alert et" gibi kuralları nadiren kullanırız. Bağımlılık temelli kurallar genellikle sistemimizin çok kararlı kısımlarına aittir, örneğin kullanıcı trafiğini datacenter'dan uzaklaştırma sistemimiz gibi. Örneğin, "Eğer datacenter drained edilmişse, latency'si için alert etme" yaygın bir datacenter alerting kuralıdır. Google'da çok az ekip karmaşık bağımlılık hiyerarşileri tutar çünkü altyapımızın sürekli refactoring oranı istikrarlıdır.

Bu bölümde açıklanan fikirlerden bazıları hala özlemsel niteliktedir: özellikle sürekli değişen sistemlerde, semptomdan root cause'lara daha hızlı geçmek için her zaman alan vardır. Bu nedenle bu bölüm monitoring sistemleri için bazı hedefler ve bu hedeflere ulaşmanın bazı yollarını belirtse de, monitoring sistemlerinin - özellikle production probleminin başlangıcından, bir insana page'lenmesi, temel triaj ve derin debugging yoluyla kritik yolun - ekipteki herkes tarafından basit ve anlaşılabilir tutulması önemlidir.

## Semptomlar ve Nedenler

Monitoring sisteminiz iki soruyu ele almalıdır: neyin bozuk olduğu ve neden?

"Neyin bozuk olduğu" semptomu belirtir; "neden" (muhtemelen ara) nedeni belirtir.

| **Semptom** | **Neden** |
|-------------|-----------|
| **HTTP 500'leri veya 404'leri serve ediyorum** | Veritabanı sunucuları bağlantıları reddediyor |
| **Response'larım yavaş** | CPU'lar bogosort tarafından aşırı yükleniyor veya Ethernet kablosu rack altında sıkışmış, kısmi paket kaybı olarak görünüyor |
| **Antarktika'daki kullanıcılar animasyonlu kedi GIF'leri almıyor** | Content Distribution Network'ünüz bilim insanlarından ve kedilerden nefret ediyor ve bu nedenle bazı client IP'leri blacklist'lemiş |
| **Private içerik dünyaca okunabilir** | Yeni bir yazılım push'ı ACL'lerin unutulmasına neden oldu ve tüm request'lere izin verdi |

"Ne" karşı "neden" maksimum sinyal ve minimum gürültü ile iyi monitoring yazmanın en önemli ayrımlarından biridir.

## Black-Box ve White-Box

White-box monitoring'i yoğun kullanımla mütevazı ama kritik black-box monitoring kullanımları birleştiriyoruz. Black-box monitoring ile white-box monitoring arasındaki farkı düşünmenin en basit yolu, black-box monitoring'in semptom odaklı olması ve tahmin edilmeyen değil aktif problemleri temsil etmesidir: "Sistem şu anda doğru çalışmıyor." White-box monitoring sistemin iç yapısını, log'lar veya HTTP endpoint'leri gibi instrumentation ile inceleme yeteneğine bağlıdır. Bu nedenle white-box monitoring yaklaşan problemlerin, retry'lar tarafından maskelenen başarısızlıkların vs. algılanmasına izin verir.

Çok katmanlı bir sistemde, bir kişinin semptomu başka bir kişinin nedenidir. Örneğin, veritabanının performansının yavaş olduğunu varsayın. Yavaş veritabanı okumaları, bunları algılayan veritabanı SRE'si için bir semptomdur. Ancak yavaş web sitesini gözlemleyen frontend SRE için aynı yavaş veritabanı okumaları bir nedendir.

## Dört Altın Sinyal

İzlemenin dört altın sinyali gecikme, trafik, hatalar ve doygunluktur. Eğer kullanıcıya yönelik sisteminizin sadece dört metriğini ölçebiliyorsanız, bu dördüne odaklanın.

**Gecikme (Latency)**

İsteği karşılamak için gereken süre. Başarılı isteklerin gecikmesi ile başarısız isteklerin gecikmesini ayırt etmek önemlidir. Örneğin, veritabanı veya diğer kritik arka sisteme bağlantı kaybı nedeniyle tetiklenen HTTP 500 hatası çok hızlı sunulabilir; ancak HTTP 500 hatası başarısız bir isteği belirttiği için, 500'leri genel gecikmenize dahil etmek yanıltıcı hesaplamalara neden olabilir. Öte yandan, yavaş bir hata hızlı bir hatadan bile daha kötüdür! Bu nedenle, sadece hataları filtrelemenin aksine, hata gecikmesini takip etmek önemlidir.

**Trafik (Traffic)**

Sisteminiz üzerindeki talep miktarının yüksek seviyeli sistem özelinde bir metrikle ölçülmesi. Web servisi için bu ölçüm genellikle saniye başına HTTP istekleridir, belki de isteklerin doğasına göre (örneğin, statik vs. dinamik içerik) ayrılır. Ses akış sistemi için bu ölçüm ağ G/Ç oranı veya eşzamanlı oturumlara odaklanabilir. Anahtar-değer depolama sistemi için bu ölçüm saniye başına işlemler ve getirmeler olabilir.

**Hatalar (Errors)**

Açık bir şekilde (örneğin, HTTP 500'ler), dolaylı olarak (örneğin, HTTP 200 başarı yanıtı, ancak yanlış içerikle birleştirilmiş) veya ilke ile (örneğin, "Bir saniye yanıt sürelerine taahhüt ettiyseniz, bir saniyenin üzerindeki herhangi bir istek bir hatadır") başarısız olan isteklerin oranı. Protokol yanıt kodlarının tüm başarısızlık koşullarını ifade etmede yetersiz olduğu durumlarda, kısmi başarısızlık modlarını takip etmek için ikincil (dahili) protokoller gerekli olabilir.

**Doygunluk (Saturation)**

Servisinizin ne kadar "dolu" olduğu. En kısıtlı kaynaklara vurgu yaparak sistem oranınızın bir ölçüsü (örneğin, bellek kısıtlı sistemde belleği göster; G/Ç kısıtlı sistemde G/Ç'yi göster). Birçok sistemin %100 kullanımı elde etmeden performansta düşüş yaşadığını unutmayın, bu nedenle kullanım hedefi belirlemek esastır.

Eğer dört altın sinyali ölçer ve bir sinyal problematik olduğunda (veya saturation durumunda, neredeyse problematik olduğunda) bir insanı page'lerseniz, servisiniz en azından monitoring tarafından makul şekilde kapsanacaktır.

## Tail Hakkında Endişelenmek (veya, Instrumentation ve Performance)

Sıfırdan bir monitoring sistemi kurarken, bir şeyin ortalamasına dayalı sistem tasarlamak cazip gelebilir: ortalama latency, node'larınızın ortalama CPU kullanımı veya veritabanlarınızın ortalama doluluk oranı. Son iki durumun sunduğu tehlike açıktır: CPU'lar ve veritabanları kolayca çok dengesiz bir şekilde kullanılabilir. Aynı durum latency için de geçerlidir. Eğer saniye başına 1.000 istek ile ortalama latency'si 100 ms olan bir web servisi çalıştırıyorsanız, isteklerin %1'i kolayca 5 saniye alabilir<sup>23</sup>. Kullanıcılarınız sayfalarını oluşturmak için birkaç böyle web servisine bağımlıysa, bir backend'in 99'uncu yüzdeliği kolayca frontend'inizin medyan response'u haline gelebilir.

Yavaş ortalama ile çok yavaş istek "tail'ini" ayırt etmenin en basit yolu, gerçek latency'ler yerine latency'lere göre gruplanmış istek sayılarını toplamaktır (histogram oluşturmaya uygun): 0 ms ile 10 ms arası, 10 ms ile 30 ms arası, 30 ms ile 100 ms arası, 100 ms ile 300 ms arası kaç istek serve ettim? Histogram sınırlarını yaklaşık olarak üssel olarak dağıtmak (bu durumda kabaca 3 faktörleriyle) genellikle isteklerinizin dağılımını görselleştirmenin kolay bir yoludur.

## Ölçümler İçin Uygun Çözünürlük Seçmek

Bir sistemin farklı yönleri farklı granülite seviyelerinde ölçülmelidir. Örneğin:

- Bir dakikanın zaman aralığında CPU yükünü gözlemlemek, yüksek tail latency'lerine yol açan oldukça uzun süreli spike'ları bile ortaya çıkarmayacaktır.
- Öte yandan, yılda 9 saatten fazla olmayan toplam downtime'ı hedefleyen web servisi için (%99.9 yıllık uptime), dakikada bir veya iki kezden fazla 200 (başarı) durumu için probe yapmak muhtemelen gereksiz sıklıktadır.

Ölçümlerinizin granülitesini nasıl yapılandırdığınızda dikkatli olun. CPU yükünün saniye başına ölçümleri ilginç veriler üretebilir, ancak bu kadar sık ölçümler toplamak, depolamak ve analiz etmek çok pahalı olabilir. Monitoring hedefiniz yüksek çözünürlük gerektiriyorsa ancak aşırı düşük latency gerektirmiyorsa, sunucuda dahili örnekleme yaparak, ardından o dağılımı zaman içinde veya sunucular arasında toplamak ve birleştirmek için harici bir sistem yapılandırarak bu maliyetleri azaltabilirsiniz. Şunları yapabilirsiniz:

1. Her saniye mevcut CPU kullanımını kaydet.
2. %5 granüliteli bucket'lar kullanarak, her saniye uygun CPU kullanım bucket'ını artır.
3. Bu değerleri her dakika birleştir.

Bu strateji, collection ve retention nedeniyle çok yüksek maliyete katlanmadan kısa CPU hotspot'larını gözlemlemenizi sağlar.

## Mümkün Olduğunca Basit, Daha Basit Değil

Tüm bu gereksinimleri üst üste yığmak çok karmaşık bir monitoring sistemine dönüşebilir - sisteminiz şu karmaşıklık seviyelerinde sonuçlanabilir:

- Farklı latency threshold'larında, farklı yüzdeliklerde, her türlü farklı metrikte alert'ler
- Olası nedenleri algılamak ve açığa çıkarmak için ekstra kod
- Bu olası nedenlerin her biri için ilişkili dashboard'lar

Potansiyel karmaşıklık kaynakları hiç bitmeyen. Tüm yazılım sistemleri gibi, monitoring kırılgan, değiştirmesi karmaşık ve bakım yükü haline gelebilir.

Bu nedenle, monitoring sisteminizi sadelik gözetmek suretiyle tasarlayın. Neyi monitor edeceğinizi seçerken aşağıdaki yönergeleri aklınızda bulundurun:

- Gerçek incident'ları en sık yakalayan kurallar mümkün olduğunca basit, öngörülebilir ve güvenilir olmalıdır.
- Nadiren test edilen (örneğin, bazı SRE ekipleri için çeyrek yılda bir kereden az) veri toplama, birleştirme ve alerting konfigürasyonu kaldırma için aday olmalıdır.
- Toplanan ancak herhangi bir önceden hazırlanmış dashboard'da açığa çıkarılmayan veya herhangi bir alert tarafından kullanılmayan sinyaller kaldırma için adaylardır.

Google'ın deneyiminde, alerting ve dashboard'lar ile eşleştirilmiş temel metrik toplama ve birleştirme, nispeten bağımsız sistem olarak iyi çalışmıştır. (Aslında Google'ın monitoring sistemi birkaç binary'ye ayrılmıştır, ancak genellikle insanlar bu binary'lerin tüm yönlerini öğrenir.) Monitoring'i karmaşık sistemleri incelemenin diğer yönleriyle - detaylı sistem profiling, tek process debugging, exception veya crash'ler hakkında detayları takip etme, load testing, log collection ve analysis veya traffic inspection - birleştirmek cazip olabilir. Bu konuların çoğu temel monitoring ile ortak noktalar paylaşsa da, çok fazla şeyi bir araya getirmek aşırı karmaşık ve kırılgan sistemlerle sonuçlanır. Software engineering'in birçok diğer yönünde olduğu gibi, net, basit, gevşek bağlı entegrasyon noktaları olan ayrı sistemleri sürdürmek daha iyi bir stratejidir (örneğin, uzun bir süre boyunca sabit kalabilecek formatta özet verileri çekmek için web API'leri kullanmak).

## Bu Prensipleri Bir Araya Getirmek

Bu bölümde tartışılan prensipler, Google SRE ekipleri içinde yaygın olarak onaylanan ve takip edilen monitoring ve alerting felsefesinde bir araya getirilebilir. Bu monitoring felsefesi biraz özlemsel olsa da, yeni bir alert yazmak veya gözden geçirmek için iyi bir başlangıç noktasıdır.

Monitoring ve alerting için kurallar oluştururken, aşağıdaki soruları sormak false positive'leri ve pager burnout'u önlemenize yardımcı olabilir<sup>24</sup>:

- Bu kural urgent, actionable ve aktif olarak veya yakın bir zamanda user-visible olan *başka türlü algılanmamış bir durumu* algılıyor mu?<sup>25</sup>
- Bu alert'i zararsız olduğunu bilerek hiç yok sayabilir miyim? Ne zaman ve neden bu alert'i yok sayabilirim ve bu senaryoyu nasıl önleyebilirim?
- Bu alert kesinlikle kullanıcıların olumsuz etkilendiğini gösteriyor mu? Kullanıcıların olumsuz etkilenmediği, drained traffic veya test deployment'ları gibi algılanabilir durumlar var mı ki bunlar filtrelenmeli?
- Bu alert'e yanıt olarak aksiyon alabilir miyim? Bu aksiyon urgent mı, yoksa sabaha kadar bekleyebilir mi? Aksiyon güvenli bir şekilde otomatikleştirilebilir mi? Bu aksiyon uzun vadeli bir düzeltme mi olacak, yoksa sadece kısa vadeli geçici çözüm mü?
- Başka insanlar bu sorun için page alıyor mu, bu nedenle page'lerden en az biri gereksiz mi?

Bu sorular page'ler ve pager'lar hakkında temel bir felsefeyi yansıtır:

- Pager her çaldığında, aciliyet duygusuyla tepki verebilmeliyim. Günde sadece birkaç kez aciliyet duygusuyla tepki verebilirim, ondan sonra yoruluyorum.
- Her page actionable olmalıdır.
- Her page yanıtı zeka gerektirmelidir. Eğer page sadece robotik yanıtı hak ediyorsa, page olmamalıdır.
- Page'ler yeni bir problem veya daha önce görülmemiş bir olay hakkında olmalıdır.

## Uzun Vade İçin Monitoring

Modern production sistemlerinde, monitoring sistemleri değişen yazılım mimarisi, yük karakteristikleri ve performans hedefleri ile sürekli gelişen sistemi takip eder. Şu anda istisnai olarak nadir ve otomatikleştirmesi zor olan bir alert sık hale gelebilir, hatta onu çözmek için aceleyle hazırlanmış bir script'i hak edebilir. Bu noktada, birisinin problemin root cause'larını bulup ortadan kaldırması gerekir; böyle bir çözüm mümkün değilse, alert yanıtı tamamen otomatikleştirilmeyi hak eder.

Monitoring hakkındaki kararların uzun vadeli hedefleri göz önünde bulundurarak alınması önemlidir. Bugün gerçekleşen her page, bir insanı yarın için sistemi geliştirmekten alıkoyar, bu nedenle sistemin uzun vadeli görünümünü iyileştirmek için kısa vadeli availability veya performance kaybını göze alma durumu sıklıkla ortaya çıkar. Bu ödünleşimi gösteren iki vaka çalışmasına bakalım.

### Bigtable SRE: Aşırı Alerting Hikayesi

Google'ın dahili altyapısı genellikle Service Level Objective (SLO) karşısında sunulur ve ölçülür. Yıllar önce, Bigtable servisinin SLO'su sentetik iyi davranışlı bir client'ın ortalama performansına dayanıyordu. Bigtable ve storage stack'inin alt katmanlarındaki problemler nedeniyle, ortalama performans büyük bir "tail" tarafından yönlendiriliyordu: isteklerin en kötü %5'i genellikle geri kalanından önemli ölçüde daha yavaştı.

SLO'ya yaklaşıldığında e-posta alert'leri tetikleniyordu ve SLO aşıldığında paging alert'leri tetikleniyordu. Her iki alert türü de yoğun şekilde ateşleniyordu ve kabul edilemez miktarda mühendislik zamanı tüketiyordu: ekip, gerçekten actionable olanları bulmak için alert'leri triaj etmek için önemli miktarda zaman harcıyordu ve kullanıcıları gerçekten etkileyen problemleri sık sık kaçırıyorduk çünkü bunlardan çok azı öyleydi. Page'lerin çoğu altyapıdaki iyi anlaşılan problemler nedeniyle acil değildi ve ya kalıplaşmış yanıtları vardı ya da hiç yanıt almıyordu.

Durumu düzeltmek için ekip üç yönlü bir yaklaşım kullandı: Bigtable'ın performansını iyileştirmek için büyük çabalar sarf ederken, aynı zamanda 75'inci yüzdelik request latency'sini kullanarak SLO hedefimizi geçici olarak geri çektik. Ayrıca, tanı koymak için zaman harcamanın uygulanamaz olduğu çok fazla olduğu için e-posta alert'lerini devre dışı bıraktık.

Bu strateji bize, sürekli taktiksel problemleri düzeltmek yerine, Bigtable ve storage stack'inin alt katmanlarındaki uzun vadeli problemleri gerçekten düzeltmek için yeterli nefes alma alanı verdi. On-call mühendisleri her saatte page'lerle uyanık tutulmadıklarında gerçekten iş başarabiliyorlardı. Sonuçta, alert'lerimizde geçici olarak geri adım atmak, daha iyi bir servise doğru daha hızlı ilerleme kaydetmemizi sağladı.

### Gmail: İnsanlardan Öngörülebilir, Script'lenebilir Yanıtlar

Gmail'in ilk günlerinde, servis Workqueue adında yeniden uyarlanmış distributed process management sistemi üzerine kuruluydu; bu sistem başlangıçta search index'in parçalarının batch işlenmesi için oluşturulmuştu. Workqueue uzun yaşam döngülü process'lere "uyarlandı" ve ardından Gmail'e uygulandı, ancak scheduler'daki nispeten opak codebase'teki belirli bug'lar yenilmesi zor oldu.

O zamanlar, Gmail monitoring'i individual task'ların Workqueue tarafından "de-schedule" edildiğinde alert'lerin ateşlendiği şekilde yapılandırılmıştı. Bu kurulum idealden uzaktı çünkü o zamanlar bile Gmail'in binlerce task'ı vardı, her task kullanıcılarımızın yüzdenin bir kısmını temsil ediyordu. Gmail kullanıcıları için iyi bir kullanıcı deneyimi sağlamayı derinden önemsiyorduk, ancak böyle bir alerting kurulumu sürdürülemezdi.

Bu problemi ele almak için Gmail SRE, kullanıcılara etkiyi minimize edecek şekilde scheduler'ı "dürtmede" yardımcı olan bir araç oluşturdu. Ekip, daha iyi uzun vadeli çözüm elde edilene kadar problemi algılamaktan rescheduler'ı dürtmeye kadar tüm döngüyü otomatikleştirmemiz gerekip gerekmediği konusunda birkaç tartışma yaptı, ancak bazıları bu tür geçici çözümün gerçek düzeltmeyi geciktireceğinden endişelendi.

Bu tür gerilim ekip içinde yaygındır ve genellikle ekibin öz disiplinine yönelik altta yatan bir güvensizliği yansıtır: bazı ekip üyeleri uygun düzeltme için zaman tanımak üzere bir "hack" implement etmek istese de, diğerleri bir hack'in unutulacağından veya uygun düzeltmenin süresiz olarak önceliğinin düşürüleceğinden endişe eder. Bu endişe güvenilirdir, çünkü problemlerin üzerini yamamak yerine gerçek düzeltmeler yapmak yerine sürdürülemez teknik borç katmanları oluşturmak kolaydır. Manager'lar ve teknik liderler, paging'in başlangıç "acısı" azalsa bile, potansiyel zaman alıcı uzun vadeli düzeltmeleri destekleyip önceliklendirerek gerçek, uzun vadeli düzeltmeleri implement etmede kilit rol oynar.

Kalıplaşmış, algoritmik yanıtları olan page'ler kırmızı bayrak olmalıdır. Ekibinizin böyle page'leri otomatikleştirme konusundaki isteksizliği, ekibin teknik borcunu temizleyebileceğine dair güven eksikliği olduğunu gösterir. Bu, escalate edilmeye değer büyük bir problemdir.

### Uzun Vade

Bigtable ve Gmail'in önceki örneklerini birleştiren ortak bir tema vardır: kısa vadeli ve uzun vadeli availability arasındaki gerilim. Çoğu zaman, saf kuvvet çabası rickety bir sistemin yüksek availability elde etmesine yardımcı olabilir, ancak bu yol genellikle kısa ömürlüdür ve burnout ve az sayıda heroik ekip üyesine bağımlılık ile doludur. Kontrollü, kısa vadeli availability düşüşü almak genellikle acı verici ama sistemin uzun vadeli kararlılığı için stratejik bir ticaret yapmaktır. Her page'i izole bir olay olarak düşünmemek, bunun yerine genel paging *seviyesinin* sağlıklı, uygun şekilde mevcut sistem ve sağlıklı, yaşayabilir ekip ve uzun vadeli görünüme yol açıp açmadığını düşünmek önemlidir. Page sıklığı hakkındaki istatistikleri (genellikle vardiya başına incident olarak ifade edilir, burada bir incident birkaç ilgili page'den oluşabilir) management ile üç aylık raporlarda gözden geçiriyoruz ve karar vericilerin pager yükü ve ekiplerinin genel sağlığı hakkında güncel tutulmalarını sağlıyoruz.

## Sonuç

Sağlıklı bir monitoring ve alerting pipeline basit ve akıl yürütülmesi kolaydır. Paging için öncelikle semptomlara odaklanır, neden odaklı heuristik'leri debugging problemlerine yardım olarak hizmet etmek üzere saklar. Semptomları monitor etmek, stack'inizde ne kadar "yukarıda" monitor ederseniz o kadar kolaydır, ancak veritabanları gibi alt sistemlerin saturation ve performansını monitor etmek genellikle doğrudan alt sistemde gerçekleştirilmelidir.

Uzun vadede, başarılı bir on-call rotasyonu ve ürün elde etmek, semptomlarda veya yakın gerçek problemlerde alert vermeyi seçmeyi, hedeflerinizi gerçekten ulaşılabilir amaçlara uyarlamayı ve monitoring'inizin hızlı teşhisi desteklediğinden emin olmayı içerir.

---

**Dipnotlar:**

*22* Bazen "alert spam" olarak bilinir, çünkü nadiren okunur veya harekete geçilir.

*23* Eğer isteklerinizin %1'i ortalamanın 50 katıysa, bu geri kalan isteklerinizin ortalamadan yaklaşık iki kat daha hızlı olduğu anlamına gelir. Ancak dağılımınızı ölçmüyorsanız, isteklerinizin çoğunun ortalamaya yakın olduğu fikri sadece ümitli düşüncedir.

*24* Başka bir bağlamda alert yorgunluğu örneği için *Applying Cardiac Alarm Management Techniques to Your On-Call* [Hol14] makalesine bakın.

*25* Sıfır-redundans (*N* + 0) durumları yakın tehlike olarak sayılır, "neredeyse dolu" servis kısımları da öyle! Redundans kavramı hakkında daha fazla detay için bkz: https://en.wikipedia.org/wiki/N%2B1_redundancy. 