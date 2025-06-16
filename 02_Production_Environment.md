# Bölüm 2 - Google'da Production Environment, Bir SRE'nin Bakış Açısından

**Yazan:** JC van Winkel  
**Editör:** Betsy Beyer

Google datacenterları çoğu geleneksel datacenter ve küçük ölçekli server farm'larından çok farklıdır. Bu farklılıklar hem ekstra problemler hem de fırsatlar sunar. Bu bölüm Google datacenterlarını karakterize eden zorlukları ve fırsatları tartışır ve kitap boyunca kullanılan terminolojiyi tanıtır.

## Hardware

Google'ın compute kaynaklarının çoğu, Google tarafından tasarlanan datacenterlarda bulunan özel power distribution, cooling, networking ve compute hardware'ı ile bulunur. "Standart" colocation datacenterların aksine, Google tasarımı bir datacenterdaki compute hardware'ı tamamı aynıdır. Server hardware'ı ile server software'ı arasındaki karışıklığı ortadan kaldırmak için, kitap boyunca şu terminolojiyi kullanıyoruz:

**Machine (Makine)**
Bir hardware parçası (veya belki bir VM)

**Server**
Bir servisi uygulayan bir software parçası

Makineler herhangi bir serveri çalıştırabilir, bu yüzden belirli makineleri belirli server programlarına tahsis etmiyoruz. Örneğin mail sunucumuzu çalıştıran belirli bir makine yoktur. Bunun yerine, kaynak tahsisi cluster işletim sistemimiz _Borg_ tarafından yönetilir.

Bu _server_ kelimesinin kullanımının alışılmadık olduğunu fark ediyoruz. Kelimenin yaygın kullanımı "network bağlantısı kabul eden binary" ile _makineyi_ birbirine karıştırır, ancak ikisini ayırt etmek Google'da bilgisayarla çalışırken önemlidir. Bizim _server_ kullanımımıza alıştığınızda, sadece Google içinde değil bu kitabın geri kalanında da bu özel terminolojiyi kullanmanın neden mantıklı olduğu daha belirgin hale gelir.

Şekil 2-1 bir Google datacenter'ının topolojisini gösterir:

* Onlarca makine bir _rack_'te yerleştirilir.
* Rack'ler bir _row_'da (sıra) durur.
* Bir veya daha fazla row bir _cluster_ oluşturur.
* Genellikle bir _datacenter_ binası birden fazla cluster barındırır.
* Yakın yerlerde bulunan birden fazla datacenter binası bir _campus_ oluşturur.

Belirli bir datacenter içindeki makinelerin birbirleriyle konuşabilmesi gerekir, bu yüzden on binlerce portlu çok hızlı bir virtual switch oluşturduk. Bunu _Jupiter_ adlı Clos network fabric'te yüzlerce Google yapımı switch'i bağlayarak başardık. En büyük konfigürasyonunda Jupiter, serverlar arasında 1.3 Pbps bisection bandwidth destekler.

Datacenterlar dünya çapındaki backbone networkümüz _B4_ ile birbirlerine bağlanır. B4 bir software-defined networking mimarisidir (ve OpenFlow açık standart iletişim protokolünü kullanır). Mütevazı sayıda siteye büyük miktarda bandwidth sağlar ve ortalama bandwidth'i maksimize etmek için elastic bandwidth allocation kullanır.

# Hardware'ı "Organize Eden" System Software'ı

Hardware'ımız büyük ölçeği kaldırabilecek software tarafından kontrol edilmeli ve yönetilmelidir. Hardware arızaları software ile yönettiğimiz önemli bir problemdir. Bir clusterdaki çok sayıda hardware komponenti göz önüne alındığında, hardware arızaları oldukça sık meydana gelir. Tipik bir yılda tek bir clusterde binlerce makine arızalanır ve binlerce hard disk bozulur; global olarak işlettiğimiz cluster sayısıyla çarpıldığında, bu sayılar biraz nefes kesici hale gelir. Bu nedenle, bu tür problemleri kullanıcılardan soyutlamak istiyoruz ve servislerimizi çalıştıran takımlar da hardware arızalarıyla uğraşmak istemez. Her datacenter campus'ında hardware ve datacenter altyapısının bakımını yapmaya adanmış takımlar vardır.

## Makineleri Yönetmek

Şekil 2-2'de gösterilen _Borg_, Apache Mesos'a benzer bir distributed cluster işletim sistemidir. Borg job'larını cluster seviyesinde yönetir.

Borg kullanıcıların _job_'larını çalıştırmaktan sorumludur; bunlar süresiz çalışan serverlar veya MapReduce gibi batch işlemler olabilir. Job'lar güvenilirlik nedenleriyle ve tek bir process genellikle tüm cluster trafiğini kaldıramadığı için birden fazla (bazen binlerce) özdeş _task_'tan oluşabilir. Borg bir job başlattığında, task'lar için makineler bulur ve makinelere server programını başlatmalarını söyler. Borg daha sonra bu task'ları sürekli izler. Eğer bir task arızalanırsa, öldürülür ve muhtemelen farklı bir makinede yeniden başlatılır.

Task'lar makineler üzerinde akışkan olarak tahsis edildiğinden, task'lara başvurmak için basitçe IP adreslerine ve port numaralarına güvenemeyiz. Bu problemi ek bir indirection seviyesiyle çözüyoruz: bir job başlatırken, Borg her task'a _Borg Naming Service_ (BNS) kullanarak bir isim ve index numarası tahsis eder. IP adresi ve port numarası kullanmak yerine, diğer processler Borg task'larına BNS ismi aracılığıyla bağlanır, bu da BNS tarafından IP adresi ve port numarasına çevrilir. Örneğin, BNS path'i `/bns/<cluster>/<user>/<job name>/<task number>` gibi bir string olabilir ve bu `<IP address>:<port>` şeklinde çözümlenir.

Borg ayrıca job'lara kaynak tahsisinden de sorumludur. Her job'ın gerekli kaynaklarını belirtmesi gerekir (örneğin, 3 CPU core, 2 GiB RAM). Tüm job'ların gereksinimlerinin listesini kullanarak, Borg task'ları makineler üzerinde failure domain'leri de hesaba katan optimal bir şekilde binpack edebilir (örneğin: Borg bir job'ın tüm task'larını aynı rack'te çalıştırmaz, çünkü bunu yapmak top of rack switch'ini o job için single point of failure yapar).

Eğer bir task talep ettiğinden daha fazla kaynak kullanmaya çalışırsa, Borg task'ı öldürür ve yeniden başlatır (yavaş crashloop yapan bir task genellikle hiç yeniden başlatılmamış bir task'tan daha iyidir).

## Storage

Task'lar makinelerdeki local disk'i scratch pad olarak kullanabilir, ancak kalıcı storage (ve hatta scratch space bile sonunda cluster storage modeline geçecek) için birkaç cluster storage seçeneğimiz var. Bunlar ikisi de açık kaynak cluster filesystem'ları olan Lustre ve Hadoop Distributed File System (HDFS) ile karşılaştırılabilir.

Storage katmanı, kullanıcılara bir cluster için mevcut storage'a kolay ve güvenilir erişim sunmaktan sorumludur. Şekil 2-3'te gösterildiği gibi, storage'ın birçok katmanı vardır:

1. En alt katman _D_ olarak adlandırılır (_disk_ için, ancak D hem spinning diskler hem de flash storage kullanır). D bir clusterdaki neredeyse tüm makinelerde çalışan bir fileserver'dır. Ancak verilerine erişmek isteyen kullanıcılar verilerinin hangi makinede saklandığını hatırlamak istemez, işte bir sonraki katmanın devreye girdiği yer burasıdır.

2. D'nin üstünde _Colossus_ adlı bir katman, olağan filesystem semantiği, replikasyon ve şifreleme sunan cluster çapında bir filesystem oluşturur. Colossus, Google File System olan GFS'nin halefidir.

3. Colossus'un üstünde birkaç database benzeri servis bulunur:
   - **Bigtable** petabyte boyutunda database'leri kaldırabilen bir NoSQL database sistemidir. Bir Bigtable row key, column key ve timestamp ile indexlenmiş sparse, distributed, persistent multidimensional sorted map'tir; map'teki her değer yorumlanmamış byte dizisidir. Bigtable eventually consistent, cross-datacenter replikasyonu destekler.
   - **Spanner** dünya çapında gerçek consistency gerektiren kullanıcılar için SQL benzeri bir arayüz sunar.
   - _Blobstore_ gibi diğer birkaç database sistemi mevcuttur. Bu seçeneklerin her biri kendi trade-off'ları ile gelir.

## Networking

Google'ın network hardware'ı çeşitli şekillerde kontrol edilir. Daha önce tartıştığımız gibi, OpenFlow tabanlı software-defined network kullanıyoruz. "Akıllı" routing hardware'ı kullanmak yerine, network boyunca en iyi yolları önceden hesaplayan merkezi (yedekli) controller ile kombinasyon halinde daha ucuz "aptal" switching komponentlerine güveniyoruz. Bu nedenle, compute-expensive routing kararlarını router'lardan uzaklaştırabilir ve basit switching hardware'ı kullanabiliriz.

Network bandwidth'inin akıllıca tahsis edilmesi gerekir. Borg'un bir task'ın kullanabileceği compute kaynaklarını sınırlaması gibi, Bandwidth Enforcer (BwE) ortalama mevcut bandwidth'i maksimize etmek için mevcut bandwidth'i yönetir.

Bazı servislerin dünya çapında dağıtılmış birden fazla clusterde çalışan job'ları vardır. Global olarak dağıtılmış servisler için latency'yi minimize etmek için, kullanıcıları mevcut kapasiteye sahip en yakın datacenter'a yönlendirmek istiyoruz. _Global Software Load Balancer_ (GSLB) üç seviyede load balancing gerçekleştirir:

* DNS istekleri için geographic load balancing (örneğin, _www.google.com_)
* User service seviyesinde load balancing (örneğin, YouTube veya Google Maps)
* Remote Procedure Call (RPC) seviyesinde load balancing

Service sahipleri bir servis için sembolik bir isim, server'ların BNS adreslerinin listesi ve her lokasyonda mevcut kapasiteyi (genellikle saniye başına sorgu ile ölçülür) belirtir. GSLB daha sonra trafiği BNS adreslerine yönlendirir.

# Diğer System Software'ları

Bir datacenterdaki diğer birkaç komponent de önemlidir.

## Lock Service

_Chubby_ lock servisi lock'ları sürdürmek için filesystem benzeri bir API sağlar. Chubby bu lock'ları datacenter lokasyonları boyunca yönetir. Asynchronous Consensus için Paxos protokolünü kullanır.

Chubby ayrıca master election'da önemli bir rol oynar. Bir servisin güvenilirlik amaçlı beş replika job'ı çalışıyor ancak sadece bir replika gerçek işi yapabiliyorsa, Chubby _hangi_ replikanın devam edebileceğini seçmek için kullanılır.

Consistent olması gereken veriler Chubby'de saklanmaya uygundur. Bu nedenle, BNS, BNS path'leri ile `IP address:port` çiftleri arasındaki mapping'i saklamak için Chubby kullanır.

## Monitoring ve Alerting

Tüm servislerin gerektiği gibi çalıştığından emin olmak istiyoruz. Bu nedenle, _Borgmon_ monitoring programımızın birçok instance'ını çalıştırıyoruz. Borgmon düzenli olarak monitörlenen serverlardan metrikleri "scrape" eder. Bu metrikler alerting için anında kullanılabilir ve ayrıca tarihi genel bakışlarda (örneğin, grafikler) kullanım için saklanabilir. Monitoring'i çeşitli şekillerde kullanabiliriz:

* Akut problemler için alerting kurma.
* Davranış karşılaştırması: bir software güncellemesi serveri daha hızlı mı yaptı?
* Kaynak tüketimi davranışının zaman içinde nasıl evrildiğini inceleme, bu capacity planning için esastır.

# Software Altyapımız

Software mimarimiz hardware altyapımızdan en verimli şekilde yararlanmak için tasarlanmıştır. Kodumız yoğun şekilde multithreaded'dir, bu yüzden bir task kolayca birçok core kullanabilir. Dashboard'lar, monitoring ve debugging'i kolaylaştırmak için, her server belirli bir task için diagnostik ve istatistik sağlayan bir HTTP server'a sahiptir.

Google'ın tüm servisleri _Stubby_ adlı Remote Procedure Call (RPC) altyapısı kullanarak iletişim kurar; açık kaynak versiyonu olan gRPC mevcuttur. Sıklıkla yerel programdaki bir subroutine çağrısı yapmak gerektiğinde bile RPC çağrısı yapılır. Bu, daha fazla modülerlik gerektiğinde veya bir server'ın codebase'i büyüdüğünde çağrıyı farklı bir server'a refactor etmeyi kolaylaştırır. GSLB, externally visible servisleri load balance ettiği şekilde RPC'leri de load balance edebilir.

Bir server _frontend_'inden RPC istekleri alır ve _backend_'ine RPC gönderir. Geleneksel terimlerle, frontend client, backend ise server olarak adlandırılır.

Veriler RPC'ye ve RPC'den Apache'nin Thrift'ine benzer olan _protocol buffer_'lar kullanılarak aktarılır, genellikle "protobuf" olarak kısaltılır. Protocol buffer'lar structured data serialize etmek için XML'e göre birçok avantaja sahiptir: kullanımı daha basit, 3 ila 10 kat daha küçük, 20 ila 100 kat daha hızlı ve daha az belirsizdir.

# Development Environment'ımız

Development velocity Google için çok önemlidir, bu yüzden altyapımızdan yararlanmak için eksiksiz bir development environment oluşturduk.

Kendi açık kaynak repository'leri olan birkaç grup dışında (örneğin, Android ve Chrome), Google Software Engineer'ları tek bir paylaşılan repository'den çalışır. Bu workflow'larımız için birkaç önemli pratik sonucu vardır:

* Engineer'lar projelerinin dışındaki bir komponente problem yaşarlarsa, problemi düzeltebilir, önerilen değişiklikleri ("changelist" veya _CL_) sahibine review için gönderebilir ve CL'yi mainline'a submit edebilir.
* Bir engineer'ın kendi projesindeki kaynak kod değişiklikleri review gerektirir. Tüm software submit edilmeden önce review edilir.

Software build edildiğinde, build isteği bir datacenterdaki build server'lara gönderilir. Birçok build server paralel olarak compile edebildiği için büyük build'ler bile hızlıca execute edilir. Bu altyapı ayrıca continuous testing için de kullanılır. Bir CL submit edildiğinde her seferinde, o CL'ye doğrudan veya dolaylı olarak bağımlı olabilecek tüm software üzerinde testler çalışır. Eğer framework değişikliğin sistemin diğer kısımlarını muhtemelen bozduğunu belirlerse, submit edilen değişikliğin sahibini bilgilendirir. Bazı projeler push-on-green sistemi kullanır; burada testleri geçtikten sonra yeni versiyon otomatik olarak production'a push edilir.

# Shakespeare: Örnek Bir Servis

Google production environment'ında bir servisin hipotetik olarak nasıl deploy edileceğinin bir modelini sağlamak için birden fazla Google teknolojisiyle etkileşime giren örnek bir servise bakalım. Diyelim ki Shakespeare'in tüm eserlerinde belirli bir kelimenin nerede kullanıldığını belirlemenizi sağlayan bir servis sunmak istiyoruz.

Bu sistemi iki kısma ayırabiliriz:

* Shakespeare'in tüm metinlerini okuyan, index oluşturan ve index'i Bigtable'a yazan batch komponenti. Bu job'ın sadece bir kez veya çok seyrek çalışması gerekir.
* End-user isteklerini yöneten application frontend'i. Bu job her zaman ayakta durur, çünkü tüm zaman dilimlerindeki kullanıcılar Shakespeare'in kitaplarında arama yapmak isteyecektir.

Batch komponenti üç aşamadan oluşan bir MapReduce'dur.

Mapping aşaması Shakespeare'in metinlerini okur ve bunları tek kelimelere böler. Bu birden fazla worker tarafından paralel olarak gerçekleştirilirse daha hızlıdır.

Shuffle aşaması tuple'ları kelimeye göre sıralar.

Reduce aşamasında, (_kelime_, _lokasyon listesi_) tuple'ı oluşturulur.

Her tuple Bigtable'da bir row'a yazılır, kelime key olarak kullanılır.

## Bir Request'in Yaşamı

Şekil 2-4 bir kullanıcının request'inin nasıl servis edildiğini gösterir: önce, kullanıcı browser'ını _shakespeare.google.com_'a yönlendirir. Karşılık gelen IP adresini elde etmek için, kullanıcının cihazı DNS server'ı ile adresi çözer (1). Bu request sonuçta GSLB ile konuşan Google'ın DNS server'ına ulaşır. GSLB bölgeler arası frontend server'lar arasındaki trafik yükünü takip ettiği için, bu kullanıcıya gönderilecek server IP adresini seçer.

Browser bu IP'deki HTTP server'a bağlanır. Bu server (Google Frontend veya GFE olarak adlandırılır) TCP bağlantısını sonlandıran bir reverse proxy'dir (2). GFE hangi servisin gerekli olduğunu arar (web search, maps veya bu durumda Shakespeare). Yine GSLB kullanarak, server mevcut bir Shakespeare frontend server'ını bulur ve o server'a HTTP request'ini içeren bir RPC gönderir (3).

Shakespeare server'ı HTTP request'ini analiz eder ve aranacak kelimeyi içeren bir protobuf oluşturur. Shakespeare frontend server'ının şimdi Shakespeare backend server'ıyla iletişim kurması gerekir: frontend server uygun ve yüklenmemiş bir backend server'ının BNS adresini elde etmek için GSLB ile iletişim kurar (4). O Shakespeare backend server'ı şimdi istenen veriyi elde etmek için bir Bigtable server'ıyla iletişim kurar (5).

Cevap reply protobuf'a yazılır ve Shakespeare backend server'ına döndürülür. Backend sonuçları içeren protobuf'ı Shakespeare frontend server'ına verir, o da HTML'i birleştirir ve cevabı kullanıcıya döndürür.

Bu tüm olay zinciri göz açıp kapayıncaya kadar—sadece birkaç yüz milisaniye içinde gerçekleşir! Birçok hareketli parça dahil olduğu için, birçok potansiel arıza noktası vardır; özellikle, arızalanan bir GSLB büyük hasar yaratabilir. Ancak, Google'ın titiz test ve dikkatli rollout politikaları, graceful degradation gibi proaktif hata kurtarma yöntemlerimizle birlikte, kullanıcılarımızın beklediği güvenilir servisi sunmamızı sağlar.

## Job ve Data Organizasyonu

Load testing backend server'ımızın yaklaşık 100 sorgu per saniye (QPS) kaldırabileceğini belirledi. Sınırlı kullanıcı kümesiyle yapılan denemeler yaklaşık 3,470 QPS peak load beklememize yol açtı, bu yüzden en az 35 task'a ihtiyacımız var. Ancak, şu düşünceler job'da en az 37 task'a, yani N+2'ye ihtiyacımız olduğu anlamına gelir:

* Güncellemeler sırasında, bir seferde bir task kullanılamaz durumda olacak, 36 task kalır.
* Task güncellemesi sırasında makine arızası meydana gelebilir, sadece 35 task kalır, peak load'ı karşılamaya yeterlidir.

Kullanıcı trafiğinin daha yakından incelenmesi peak kullanımımızın global olarak dağıldığını gösterir: Kuzey Amerika'dan 1,430 QPS, Güney Amerika'dan 290, Avrupa ve Afrika'dan 1,400, Asya ve Avustralya'dan 350. Tüm backend'leri tek bir sitede konumlandırmak yerine, bunları ABD, Güney Amerika, Avrupa ve Asya'ya dağıtıyoruz. Bölge başına N+2 redundancy'ye izin vermek ABD'de 17 task, Avrupa'da 16 ve Asya'da 6 task ile sonuçlanır. Ancak, N+2'nin N+1'e olan overhead'ini düşürmek için Güney Amerika'da (5 yerine) 4 task kullanmaya karar veriyoruz.

Backend'lerin veriyi tutan Bigtable ile iletişim kurması gerektiği için, bu storage elementini de stratejik olarak tasarlamamız gerekir. Asya'daki bir backend'in ABD'deki Bigtable ile iletişim kurması önemli miktarda latency ekler, bu yüzden Bigtable'ı her bölgede replika ediyoruz. Bigtable replikasyonu iki şekilde bize yardımcı olur: Bigtable server'ı arızalanırsa resilience sağlar ve veri erişim latency'sini düşürür.

**Chapter 2 tamamlandı!** 🎉

Bu bölümde Google'ın production environment'ının detaylarını, Borg cluster management sistemini, storage katmanlarını (D, Colossus, Bigtable, Spanner), networking altyapısını (Jupiter, B4), monitoring sistemlerini (Borgmon) ve Shakespeare örnek servisi üzerinden bir request'in yaşam döngüsünü öğrendik.

Şimdi Chapter 3'e geçebilirim. Hazır mısın? 