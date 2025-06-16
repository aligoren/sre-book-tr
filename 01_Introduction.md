# Bölüm 1 - Giriş

**Yazan:** Benjamin Treynor Sloss  
**Editör:** Betsy Beyer

> Hope is not a strategy.
> 
> Geleneksel SRE söyleyişi

Sistemlerin kendilerini çalıştırmadığı genel kabul görmüş bir gerçektir. Peki o halde bir sistem—özellikle de büyük ölçekte çalışan karmaşık bir bilgisayar sistemi—nasıl çalıştırılmalıdır?

# Servis Yönetiminde Sysadmin Yaklaşımı

Tarihsel olarak, şirketler karmaşık bilgisayar sistemlerini çalıştırmak için system administratorleri istihdam etmişlerdir.

Bu system administrator veya sysadmin yaklaşımı, mevcut software componentlerini bir araya getirip bunları birlikte çalışacak şekilde deploy ederek bir servis üretmeyi kapsar. Daha sonra sysadminler bu servisi çalıştırmak ve olaylar ile güncellemeler meydana geldikçe bunlara yanıt vermekle görevlendirilir. Sistem karmaşıklık ve trafik hacmi açısından büyüdükçe, buna karşılık gelen olay ve güncelleme artışı meydana gelir, sysadmin ekibi de ek işi absorbe etmek için büyür. Sysadmin rolü, ürünün developerlarından belirgin şekilde farklı bir skill setine ihtiyaç duyduğu için, developerlar ve sysadminler ayrı ekiplere bölünür: "development" ve "operations" veya "ops."

Servis yönetiminin sysadmin modelinin çeşitli avantajları vardır. Bir servisi nasıl çalıştıracaklarına ve personel atayacaklarına karar veren şirketler için bu yaklaşım uygulaması nispeten kolaydır: tanıdık bir endüstri paradigması olarak, öğrenecek ve taklit edecek birçok örnek vardır. İlgili bir talent pool zaten yaygın olarak mevcuttur. Bu birleştirilmiş sistemleri çalıştırmaya yardımcı olacak mevcut araçlar, software componentleri (hazır veya başka türlü) ve entegrasyon şirketlerinin bir dizisi mevcuttur, böylece acemi bir sysadmin ekibinin tekerleği yeniden icat etmesi ve sıfırdan bir sistem tasarlaması gerekmez.

Sysadmin yaklaşımı ve ona eşlik eden development/ops ayrımının bir dizi dezavantajı ve tuzağı vardır. Bunlar genel olarak iki kategoriye ayrılır: direkt maliyetler ve dolaylı maliyetler.

Direkt maliyetler ne ince ne de belirsizdir. Hem değişiklik yönetimi hem de olay ele alma için manuel müdahaleye dayanan bir ekiple servis çalıştırmak, servis ve/veya servise trafik büyüdükçe pahalı hale gelir, çünkü ekibin boyutu zorunlu olarak sistem tarafından üretilen yükle ölçeklenmek zorundadır.

Development/ops ayrımının dolaylı maliyetleri ince olabilir, ancak çoğu zaman organizasyon için direkt maliyetlerden daha pahalıdır. Bu maliyetler, iki ekibin geçmiş, skill set ve teşvikler açısından oldukça farklı olması gerçeğinden kaynaklanır. Durumları tanımlamak için farklı kelime dağarcıkları kullanırlar; hem risk hem de teknik çözümler için olasılıklar hakkında farklı varsayımlar taşırlar; ürün istikrarının hedef seviyesi hakkında farklı varsayımları vardır. Gruplar arasındaki ayrım kolayca sadece teşvikler değil, aynı zamanda iletişim, hedefler ve sonuçta güven ve saygı ayrımı haline gelebilir. Bu sonuç bir patolojidir.

Geleneksel operations ekipleri ve ürün geliştirmede karşılıkları bu nedenle çoğu zaman çatışma içinde olurlar, en görünür şekilde software'in production'a ne kadar hızlı release edilebileceği konusunda. Özlerinde, development ekipleri yeni özellikler başlatmak ve bunların kullanıcılar tarafından benimsendiğini görmek isterler. Özlerinde ops ekipleri ise pager'ı tutarlarken servisin bozulmamasını sağlamak isterler. Çoğu outage bir tür değişiklik nedeniyle gerçekleştiği için—yeni bir konfigürasyon, yeni bir özellik lansmanı veya yeni bir tür kullanıcı trafiği—iki ekibin hedefleri temelde gerilim içindedir.

Her iki grup da çıkarlarını en çıplak şekilde ifade etmenin kabul edilemez olduğunu anlar ("Herhangi bir şeyi, herhangi bir zamanda, engel olmadan başlatmak istiyoruz" karşısında "Sistem çalışmaya başladığında asla hiçbir şeyi değiştirmek istemeyiz"). Ve kelime dağarcıkları ve risk varsayımları farklı olduğu için, her iki grup da çoğu zaman çıkarlarını ilerletmek için tanıdık bir siper savaşı biçimine başvurur. Ops ekibi değişiklik riskine karşı çalışan sistemi korumaya çalışır ve lansman ile değişiklik kapıları sunar. Örneğin, lansman incelemeleri geçmişte kesinti yaratan her problem için açık bir kontrol içerebilir—bu rastgele uzun bir liste olabilir ve tüm elementler eşit değer sağlamayabilir. Dev ekibi hızla nasıl yanıt vereceğini öğrenir. Daha az "lansman" ve daha fazla "flag flip," "incremental update" veya "cherrypick" yaparlar. Ürünü parçalayarak daha az özelliğin lansman incelemesine tabi olması gibi taktikleri benimserler.

# Google'ın Servis Yönetimi Yaklaşımı: Site Reliability Engineering

Çatışma, software servisi sunmanın kaçınılmaz bir parçası değildir. Google sistemlerimizi farklı bir yaklaşımla çalıştırmayı seçti: Site Reliability Engineering ekiplerimiz ürünlerimizi çalıştırmak ve sysadminler tarafından genellikle manuel olarak gerçekleştirilen işi başaracak sistemler oluşturmak için software engineerları işe almaya odaklanır.

Google'da tanımlandığı şekliyle Site Reliability Engineering tam olarak nedir? Açıklamam basittir: SRE, bir software engineer'den bir operations ekibi tasarlamasını istediğinizde olan şeydir. 2003'te Google'a katıldığımda ve yedi engineerdan oluşan bir "Production Team"i çalıştırmakla görevlendirildiğimde, o ana kadarki tüm hayatım software engineering olmuştu. Bu yüzden grubu kendim bir SRE olarak çalışsaydım nasıl çalışmasını isteyeceğim şekilde tasarladım ve yönettim. O grup o zamandan beri olgunlaştı ve Google'ın günümüzdeki SRE ekibi haline geldi, yaşam boyu software engineer tarafından öngörüldüğü şekliyle kökenlerine sadık kalarak.

Google'ın servis yönetimi yaklaşımının temel yapı taşı, her SRE ekibinin kompozisyonudur. Bütün olarak, SRE'ler iki ana kategoriye ayrılabilir.

%50-60'ı Google Software Engineer'ları veya daha doğrusu, Google Software Engineer'ları için standart prosedür üzerinden işe alınmış kişilerdir. Diğer %40-50'si ise Google Software Engineering niteliklerine çok yakın olan (yani gerekli skill setinin %85-99'una sahip) ve ek olarak SRE için kullanışlı ancak çoğu software engineer için nadir olan teknik becerilere sahip adaylardır. Açık farkla, UNIX sistem internalleri ve networking (Layer 1'den Layer 3'e) expertise'i aradığımız alternatif teknik becerilerin en yaygın iki türüdür.

Tüm SRE'ler için ortak olan, karmaşık problemleri çözmek için software sistemleri geliştirme konusunda inanç ve yetenek sahibi olmaktır. SRE içinde, her iki grubun kariyer ilerlemesini yakından takip ediyoruz ve bugüne kadar iki track'ten gelen engineerlar arasında performansta pratik bir fark bulmadık. Aslında, SRE ekibinin biraz çeşitli geçmişi sıklıkla birkaç skill setinin sentezinin ürünü olan zekice, yüksek kaliteli sistemlerle sonuçlanır.

SRE için işe alma yaklaşımımızın sonucu, (a) görevleri elle gerçekleştirmekten hızla sıkılacak ve (b) çözüm karmaşık olsa bile önceden manuel çalışmalarını software ile değiştirmek için gerekli skill setine sahip insanlardan oluşan bir ekiple sonuçlanmamızdır. SRE'ler ayrıca geliştirme organizasyonunun geri kalanıyla akademik ve entelektüel geçmişi paylaşırlar. Bu nedenle, SRE temelde tarihsel olarak bir operations ekibi tarafından yapılan işi yapıyor, ancak software expertise'i olan engineerları kullanıyor ve bu engineerların doğası gereği hem insan emeğini değiştirmek için software ile otomasyonu tasarlama ve uygulama konusunda hem yatkın hem de yetenekli oldukları gerçeğine güveniyor.

Tasarım gereği, SRE ekiplerinin engineering'e odaklanması kritiktir. Sürekli engineering olmadan, operations yükü artar ve ekipler workload'la yetişebilmek için sadece daha fazla insana ihtiyaç duyar. Sonuçta, geleneksel ops-odaklı bir grup servis boyutuyla lineer ölçeklenir: servis tarafından desteklenen ürünler başarılı olursa, operational yük trafikle birlikte büyür. Bu aynı görevleri tekrar tekrar yapmak için daha fazla insan işe alma anlamına gelir.

Bu kaderi önlemek için, bir servisi yönetmekle görevli ekibin kod yazması gerekir yoksa boğulur. Bu nedenle, Google tüm SRE'ler için "ops" işinin toplamında %50 bir üst sınır koyar—ticket'lar, on-call, manuel görevler, vb. Bu üst sınır SRE ekibinin programlarında servisi istikrarlı ve işletilebilir hale getirmek için yeterli zaman olmasını sağlar. Bu bir üst sınırdır; zamanla, kendi hallerine bırakıldığında, SRE ekibi çok az operational yüke sahip olmalı ve neredeyse tamamen geliştirme görevleriyle meşgul olmalıdır, çünkü servis temelde kendini çalıştırır ve onarır: otomatik olan değil, sadece otomatikleştirilmiş sistemler istiyoruz. Pratikte, ölçek ve yeni özellikler SRE'leri tetikte tutar.

Google'ın genel kuralı, bir SRE ekibinin kalan %50 zamanını gerçekte geliştirme yaparak geçirmesi gerektiğidir. Peki bu eşiği nasıl uygularız? İlk olarak, SRE zamanının nasıl harcandığını ölçmemiz gerekir. Bu ölçüm elimizdeyken, zamanlarının %50'sinden azını geliştirme işine harcayan ekiplerin uygulamalarını değiştirmelerini sağlarız. Bu çoğu zaman operations yükünün bir kısmını development ekibine geri kaydırmak veya ekibe ek operational sorumluluklar atamadan personel eklemek anlamına gelir. Bu ops ve development işi arasındaki dengeyi bilinçli olarak korumak, SRE'lerin yaratıcı, özerk engineering'de meşgul olma bandwidth'ine sahip olmalarını sağlarken aynı zamanda bir servis çalıştırmanın operations tarafından elde edilen bilgeliği korumaları mümkün kılar.

Google SRE'nin büyük ölçekli sistemleri çalıştırma yaklaşımının birçok avantajı olduğunu gördük. SRE'ler Google'ın sistemlerini kendilerini çalıştırma arayışında doğrudan kodu değiştirdikleri için, SRE ekipleri hem hızlı inovasyon hem de büyük değişiklik kabulü ile karakterize edilir. Bu tür ekipler nispeten ucuzdur—aynı servisi ops-odaklı bir ekiple desteklemek önemli ölçüde daha fazla sayıda insana ihtiyaç duyar. Bunun yerine, bir sistemi çalıştırmak, sürdürmek ve geliştirmek için gereken SRE sayısı sistemin boyutuyla sublineer ölçeklenir. Son olarak, SRE sadece dev/ops ayrımının işlevsel bozukluğunu atlatmakla kalmaz, aynı zamanda ürün geliştirme ekiplerimizi de geliştirir: ürün geliştirme ve SRE ekipleri arasındaki kolay transferler tüm grubu çapraz eğitir ve aksi takdirde milyonluk çekirdekli dağıtık sistem oluşturmayı öğrenmekte zorluk çekebilecek developerlerin becerilerini geliştirir.

Bu net kazanımlara rağmen, SRE modeli kendine özgü zorluklar setiyle karakterize edilir. Google'ın karşılaştığı sürekli bir zorluk SRE işe almaktır: SRE sadece ürün geliştirme işe alma pipeline'ı ile aynı adaylar için rekabet etmekle kalmaz, aynı zamanda hem kodlama hem de sistem engineering becerileri açısından işe alma çıtasını bu kadar yüksek koymamız gerçeği işe alma havuzumuzun zorunlu olarak küçük olduğu anlamına gelir. Disiplinimiz nispeten yeni ve benzersiz olduğu için, SRE ekibi oluşturma ve yönetme konusunda fazla endüstri bilgisi mevcut değildir (umarım bu kitap bu yönde adımlar atar!). Ve bir SRE ekibi yerleştirildikten sonra, servis yönetimine potansiyel olarak alışılmadık yaklaşımları güçlü yönetim desteği gerektirir. Örneğin, error budget tükendiğinde çeyreğin geri kalanı için release'leri durdurma kararı, yönetimleri tarafından zorunlu kılınmadığı sürece ürün geliştirme ekibi tarafından benimsenmeyebilir.

# DevOps mu SRE mi?

"DevOps" terimi endüstride 2008'in sonlarında ortaya çıktı ve bu yazım itibariyle (2016 başları) hâlâ akışkan bir durumdadır. Temel ilkeleri—sistem tasarımı ve geliştirmesinin her aşamasında BT fonksiyonunun dahil olması, insan çabasına karşı otomasyona ağır bağımlılık, engineering uygulamaları ve araçlarının operations görevlerine uygulanması—SRE'nin birçok ilke ve uygulamasıyla tutarlıdır. DevOps'u SRE'nin bazı temel ilkelerinin daha geniş organizasyon, yönetim yapıları ve personel yelpazesine genelleştirilmesi olarak görebilirsiniz. Eşit şekilde SRE'yi bazı kendine özgü uzantılarla DevOps'un belirli bir uygulaması olarak da görebilirsiniz.

# SRE'nin İlkeleri

Workflow'lar, öncelikler ve günlük operasyonların nüansları SRE ekibinden SRE ekibine değişse de, hepsi destekledikleri servis(ler) için aynı temel sorumlulukları paylaşır ve aynı temel ilkelere bağlıdır. Genel olarak, bir SRE ekibi servis(ler)inin availability, latency, performance, efficiency, change management, monitoring, emergency response ve capacity planning'inden sorumludur. SRE ekiplerinin çevreleriyle—sadece production ortamı değil, aynı zamanda ürün geliştirme ekipleri, test ekipleri, kullanıcılar vb.—nasıl etkileşime girdiğine dair engagement kuralları ve ilkeleri kodladık. Bu kurallar ve çalışma uygulamaları operations işine karşı engineering işine odaklanmamızı korumamıza yardımcı olur.

Aşağıdaki bölüm Google SRE'nin temel ilkelerinin her birini tartışır.

## Engineering'e Dayanıklı Odak Sağlama

Halihazırda tartışıldığı gibi, Google SRE'ler için operational işi zamanlarının %50'siyle sınırlar. Kalan zamanlarını coding becerilerini proje işinde kullanarak geçirmelidirler. Pratikte bu, SRE'ler tarafından yapılan operational iş miktarını izleyerek ve fazla operational işi ürün geliştirme ekiplerine yönlendirerek başarılır: bug'ları ve ticket'ları development manager'lara yeniden atama, developer'ları on-call pager rotasyonlarına (yeniden) entegre etme, vb. Yönlendirme operational yük %50 veya altına düştüğünde sona erer. Bu aynı zamanda etkili bir feedback mekanizması sağlar, developer'ları manuel müdahale gerektirmeyen sistemler oluşturmaya yönlendirir. Bu yaklaşım tüm organizasyon—SRE ve development'ın—güvenlik valfi mekanizmasının neden var olduğunu anladığında ve ürün yeterli operational yük üretmediği için taşma olayları olmaması hedefini desteklediğinde iyi çalışır.

Operations işine odaklandıklarında, ortalama olarak SRE'ler 8-12 saatlik on-call vardiya başına maksimum iki olay almalıdır. Bu hedef hacim, on-call engineer'ın olayı doğru ve hızlı şekilde ele alması, temizlik yapıp normal servisi restore etmesi ve sonra postmortem yapması için yeterli zaman verir. On-call vardiya başına düzenli olarak ikiden fazla olay meydana gelirse, problemler iyice araştırılamaz ve engineerlar bu olaylardan öğrenmelerini engelleyecek kadar bunalmış olurlar. Pager yorgunluğu senaryosu da ölçekle iyileşmez. Tersine, on-call SRE'ler tutarlı şekilde vardiya başına birden az olay alırlarsa, onları hazır tutmak zaman kaybıdır.

Postmortemlar, pager'ı tetikleyip tetiklemediğine bakılmaksızın tüm önemli olaylar için yazılmalıdır; pager tetiklemeyen postmortemlar daha da değerlidir, çünkü muhtemelen açık monitoring boşluklarına işaret ederler. Bu araştırma ne olduğunu detaylı olarak belirlemelidir, olayın tüm kök nedenlerini bulmalı ve problemi düzeltmek veya bir dahaki sefere nasıl ele alınacağını geliştirmek için eylemleri atamalıdır. Google hatayı önlemek veya minimize etmekten ziyade hataları açığa çıkarmak ve bu hataları düzeltmek için engineering uygulamak hedefiyle blame-free postmortem kültürü altında çalışır.

## Servisin SLO'sunu İhlal Etmeden Maksimum Değişiklik Hızını Takip Etme

Ürün geliştirme ve SRE ekipleri ilgili hedeflerindeki yapısal çatışmayı ortadan kaldırarak üretken bir çalışma ilişkisi kurabilirler. Yapısal çatışma inovasyon hızı ile ürün istikrarı arasındadır ve daha önce açıklandığı gibi, bu çatışma çoğu zaman dolaylı olarak ifade edilir. SRE'de bu çatışmayı öne çıkarırız ve sonra error budget sunumuyla çözeriz.

Error budget, temelde her şey için %100'ün yanlış güvenilirlik hedefi olduğu gözleminden kaynaklanır (kalp pilleri ve anti-lock frenler önemli istisnalar olmakla). Genel olarak, herhangi bir software servisi veya sistemi için, %100 doğru güvenilirlik hedefi değildir çünkü hiçbir kullanıcı %100 available olan sistem ile %99.999 available olan sistem arasındaki farkı söyleyemez. Kullanıcı ve servis arasındaki yolda birçok başka sistem vardır (laptop'ları, ev WiFi'ları, ISP'leri, elektrik şebekesi...) ve bu sistemler toplu olarak %99.999 availability'den çok daha azdır. Bu nedenle, %99.999 ile %100 arasındaki marjinal fark diğer unavailability gürültüsünde kaybolur ve kullanıcı o son %0.001 availability'yi eklemek için gereken muazzam çabadan hiçbir fayda görmez.

%100 bir sistem için yanlış güvenilirlik hedefiyse, o halde sistem için doğru güvenilirlik hedefi nedir? Bu aslında hiç teknik bir soru değildir—bu bir ürün sorusudur ve aşağıdaki hususları göz önünde bulundurmalıdır:

* Kullanıcılar ürünü nasıl kullandıkları göz önüne alındığında hangi availability seviyesinden memnun olacaklar?
* Ürünün availability'sinden memnun olmayan kullanıcılara hangi alternatifler mevcut?
* Farklı availability seviyelerinde kullanıcıların ürün kullanımına ne olur?

İşletme veya ürün sistemin availability hedefini belirlemelidir. Bu hedef belirlendikten sonra, error budget bir eksi availability hedefidir. %99.99 available olan bir servis %0.01 unavailable'dır. İzin verilen o %0.01 unavailability servisin error budget'ıdır. Budget'ı aşmadığımız sürece istediğimiz her şeye harcayabiliriz.

Peki error budget'ı nasıl harcamak istiyoruz? Development ekibi özellikler başlatmak ve yeni kullanıcıları çekmek istiyor. İdeal olarak, tüm error budget'ımızı onları hızla başlatabilmek için başlattığımız şeylerle risk alarak harcarız. Bu temel öncül error budgetların tüm modelini açıklar. SRE aktiviteleri bu framework'te kavramsallaştırıldığı anda, phased rolloutlar ve %1 deneyler gibi taktikler aracılığıyla error budget'ı serbest bırakmak daha hızlı başlatmalar için optimize edebilir.

Error budget kullanımı development ve SRE arasındaki teşvik yapısal çatışmasını çözer. SRE'nin hedefi artık "sıfır kesinti" değildir; bunun yerine, SRE'ler ve ürün developerleri maksimum özellik hızı elde etmek için error budget harcamayı hedefler. Bu değişiklik tüm farkı yaratır. Kesinti artık "kötü" bir şey değildir—inovasyon sürecinin beklenen bir parçasıdır ve hem development hem de SRE ekiplerinin korktuğu değil yönettiği bir olaydır.

## Monitoring

Monitoring, servis sahiplerinin sistemin sağlığı ve availability'sini takip etmesinin birincil araçlarından biridir. Bu nedenle, monitoring stratejisi düşünerek inşa edilmelidir. Monitoring'e klasik ve yaygın bir yaklaşım belirli bir değeri veya koşulu izlemek ve sonra o değer aşıldığında veya o koşul oluştuğunda email alert tetiklemektir. Ancak, bu tür email alerting etkili bir çözüm değildir: bir insanın email okuması ve herhangi bir eylem türünün gerekip gerekmediğine karar vermesi gereken bir sistem temelde hatalıdır. Monitoring asla alerting domain'inin herhangi bir kısmını yorumlamak için insan gerektirmemelidir. Bunun yerine, software yorumlamayı yapmalı ve insanlar sadece eylem almaları gerektiğinde bilgilendirilmelidir.

Üç tür geçerli monitoring çıktısı vardır:

**Alertler**

Bir insanın durumu iyileştirmek için olan veya olmak üzere olan bir şeye yanıt olarak hemen eylem alması gerektiğini işaret eder.

**Ticket'lar**

Bir insanın eylem alması gerektiğini ancak hemen değil işaret eder. Sistem durumu otomatik olarak ele alamaz, ancak bir insan birkaç gün içinde eylem alırsa hiçbir hasar olmaz.

**Logging**

Kimsenin bu bilgilere bakması gerekmez, ancak diagnostic veya forensic amaçlarla kaydedilir. Beklenti, başka bir şey onları buna yönlendirmedikçe kimsenin logları okumamasıdır.

## Emergency Response

Güvenilirlik ortalama başarısızlık süresi (MTTF) ve ortalama onarım süresi (MTTR) fonksiyonudur. Emergency response etkinliğini değerlendirmede en ilgili metrik, response ekibinin sistemi ne kadar hızla sağlığa döndürebildiğidir—yani MTTR.

İnsanlar latency ekler. Belirli bir sistem daha fazla gerçek başarısızlık yaşasa bile, insan müdahalesini gerektiren acil durumları önleyebilen bir sistem, hands-on müdahale gerektiren sistemden daha yüksek availability'ye sahip olacaktır. İnsanlar gerekli olduğunda, "playbook"ta en iyi uygulamaları önceden düşünüp kaydetmenin "kanat çırpmaya" kıyasla MTTR'de yaklaşık 3x iyileşme sağladığını gördük. Çok yönlü kahraman on-call engineeri işe yarar, ancak playbook ile donanmış uygulamalı on-call engineer çok daha iyi çalışır. Ne kadar kapsamlı olursa olsun hiçbir playbook, fly'da düşünebilen akıllı engineerların yerini tutmaz, ancak açık ve kapsamlı troubleshooting adımları ve ipuçları yüksek bahisli veya zamana duyarlı pager'a yanıt verirken değerlidir. Bu nedenle, Google SRE on-call playbook'lara güvenir ve engineerleri on-call olaylara tepki vermek için hazırlamak amacıyla "Wheel of Misfortune" gibi alıştırmalara ek olarak.

## Change Management

SRE kesintilerin yaklaşık %70'inin canlı sistemdeki değişiklikler nedeniyle olduğunu bulmuştur. Bu alandaki en iyi uygulamalar aşağıdakileri başarmak için otomasyonu kullanır:

* Progressive rolloutları uygulama
* Problemleri hızlı ve doğru şekilde tespit etme
* Problemler ortaya çıktığında değişiklikleri güvenle geri alma

Bu üçlü uygulama kötü değişikliklere maruz kalan kullanıcı ve işlemlerinin toplam sayısını etkili şekilde minimize eder. İnsanları döngüden çıkararak, bu uygulamalar yorgunluk, aşinalık/küçümseme ve yüksek tekrarlayan görevlere dikkatsizlik gibi normal problemleri önler. Sonuç olarak, hem release hızı hem de güvenlik artar.

## Talep Tahmini ve Capacity Planning

Talep tahmini ve capacity planning, gerekli availability ile gelecekteki öngörülen talebi karşılamak için yeterli capacity ve redundancy olduğunu sağlamak olarak görülebilir. Bu kavramlar hakkında özellikle özel bir şey yoktur, yalnızca şaşırtıcı sayıda servis ve ekibin gerekli capacity'nin ihtiyaç duyulduğu zamana kadar yerinde olduğunu sağlamak için gerekli adımları atmadığıdır. Capacity planning hem organik büyümeyi (müşteriler tarafından doğal ürün benimseme ve kullanımından kaynaklanan) hem de inorganik büyümeyi (özellik başlatmaları, pazarlama kampanyaları veya diğer iş-odaklı değişikliklerden kaynaklanan olaylar gibi) hesaba katmalıdır.

Capacity planning'de birkaç adım zorunludur:

* Capacity elde etmek için gereken lead time'ı aşan doğru organik talep tahmini
* İnorganik talep kaynaklarının talep tahminine doğru dahil edilmesi
* Ham capacity'yi (serverlar, diskler vb.) servis capacity'sine ilişkilendirmek için sistemin düzenli load testi

Capacity availability için kritik olduğu için, doğal olarak SRE ekibinin capacity planning'den sorumlu olması gerekir, bu da onların aynı zamanda provisioning'den de sorumlu olması gerektiği anlamına gelir.

## Provisioning

Provisioning hem change management hem de capacity planning'i birleştirir. Deneyimimizde, provisioning hızla ve sadece gerekli olduğunda yapılmalıdır, çünkü capacity pahalıdır. Bu alıştırma aynı zamanda doğru yapılmalıdır yoksa capacity gerektiğinde çalışmaz. Yeni capacity eklemek çoğu zaman yeni instance veya lokasyon başlatmayı, mevcut sistemlerde önemli değişiklik yapmayı (konfigürasyon dosyaları, load balancer'lar, networking) ve yeni capacity'nin performans gösterip doğru sonuçlar verdiğini doğrulamayı kapsar. Bu nedenle, saatte birkaç kez yapılan load shifting'den daha riskli bir operasyondur ve buna karşılık gelen derecede ekstra dikkat ile ele alınmalıdır.

## Efficiency ve Performance

Kaynakların verimli kullanımı bir servis para ile ilgilendiği her zaman önemlidir. SRE sonuçta provisioning'i kontrol ettiği için, utilization'la ilgili herhangi bir çalışmaya dahil olmalıdır, çünkü utilization belirli bir servisin nasıl çalıştığı ve nasıl provision edildiğinin bir fonksiyonudur. Bir servis için provisioning stratejisine ve dolayısıyla utilization'ına yakın dikkat etmenin servisin toplam maliyetlerinde çok çok büyük bir kaldıraç sağladığı sonucu çıkar.

Kaynak kullanımı talep (load), capacity ve software efficiency'nin bir fonksiyonudur. SRE'ler talebi tahmin eder, capacity provision eder ve software'i değiştirebilir. Bu üç faktör servisin efficiency'sinin büyük bir kısmıdır (tamamı olmasa da).

Software sistemleri kendilerine load eklendiğinde yavaşlar. Bir servisteki yavaşlama capacity kaybına eşittir. Bir noktada, yavaşlayan sistem servis etmeyi durdurur, bu da sonsuz yavaşlığa karşılık gelir. SRE'ler belirli bir response hızında capacity hedefini karşılamak için provision eder ve bu nedenle servisin performansıyla yakından ilgilenir. SRE'ler ve ürün developerleri performansını artırmak için servisi izler ve değiştirir (ve değiştirmeli), böylece capacity ekler ve efficiency'yi artırır.

# Başlangıcın Sonu

Site Reliability Engineering büyük, karmaşık servisleri yönetmek için mevcut endüstri en iyi uygulamalarından önemli bir kopuşu temsil eder. Başlangıçta aşinalık tarafından motive edilmiş—"bir software engineer olarak, tekrarlayan görevler setini başarmak için zamanımı böyle yatırım yapmak isterdim"—çok daha fazlası haline gelmiştir: bir ilkeler seti, bir uygulamalar seti, bir teşvikler seti ve daha büyük software engineering disiplini içinde bir çaba alanı. Kitabın geri kalanı SRE Way'i detayıyla keşfeder. 