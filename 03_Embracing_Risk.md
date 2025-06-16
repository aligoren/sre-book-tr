# Bölüm 3 - Riski Kucaklamak

**Yazan:** Marc Alvidrez  
**Editör:** Kavita Guliani

Google'ın %100 güvenilir servisler—asla arızalanmayan servisler—yapmaya çalıştığını bekleyebilirsiniz. Ancak belirli bir noktadan sonra güvenilirliği artırmanın bir servis (ve kullanıcıları) için daha iyi değil daha kötü olduğu ortaya çıkıyor! Aşırı güvenilirlik bir maliyete gelir: stability'yi maksimize etmek yeni özelliklerin ne kadar hızlı geliştirilebileceğini ve ürünlerin kullanıcılara ne kadar hızlı teslim edilebileceğini sınırlar ve maliyetlerini dramatik şekilde artırır, bu da bir takımın sunabileceği özellik sayısını azaltır. Dahası, kullanıcılar genellikle bir serviste yüksek güvenilirlik ile aşırı güvenilirlik arasındaki farkı fark etmezler, çünkü user experience cellular network veya çalıştıkları cihaz gibi daha az güvenilir komponentler tarafından dominante edilir. Basitçe söylemek gerekirse, %99 güvenilir bir smartphone kullanan bir kullanıcı %99.99 ve %99.999 servis güvenilirliği arasındaki farkı anlayamaz! Bunu göz önünde bulundurarak, Site Reliability Engineering basitçe uptime'ı maksimize etmek yerine, unavailability riskini hızlı inovasyon ve verimli servis operasyonları hedefleriyle dengelemeye çalışır, böylece kullanıcıların genel memnuniyeti—özellikler, servis ve performans ile—optimize edilir.

## Risk'i Yönetmek

Güvenilmez sistemler kullanıcıların güvenini hızla aşındırabilir, bu yüzden sistem arızası şansını azaltmak istiyoruz. Ancak deneyim gösteriyor ki sistemler inşa ederken maliyet güvenilirlik artışlarıyla linear olarak artmaz—güvenilirlikte incremental bir iyileştirme önceki artıştan 100 kat daha pahalıya mal olabilir. Maliyetlilik iki boyuta sahiptir:

**Redundant machine/compute kaynaklarının maliyeti**

Örneğin, sistemleri rutin veya öngörülemeyen bakım için offline almamızı sağlayan veya minimum veri dayanıklılığı garantisi sağlayan parity code block'larını saklamamız için yer sağlayan redundant ekipmanla ilişkili maliyet.

**Opportunity cost**

Bir organizasyonun risk azaltan sistemler veya özellikler inşa etmek için engineering kaynaklarını tahsis ettiğinde, end user'lar tarafından doğrudan görülebilen veya kullanılabilen özellikler yerine katlandığı maliyet. Bu engineer'lar artık end user'lar için yeni özellikler ve ürünler üzerinde çalışmıyor.

SRE'de servis güvenilirliğini büyük ölçüde risk'i yöneterek yönetiyoruz. Risk'i bir continuum olarak kavramsallaştırıyoruz. Google sistemlerine daha fazla güvenilirlik enjekte etmeyi ve çalıştırdığımız servisler için uygun tolerans seviyesini belirlemeyi eşit önemde görüyoruz. Bunu yapmak örneğin Search, Ads, Gmail veya Photos'u (linear olmayan) risk continuum'unda nereye yerleştirmemiz gerektiğini belirlemek için cost/benefit analizi yapmamızı sağlar. Hedefimiz belirli bir servisin aldığı risk'i işletmenin katlanmaya istekli olduğu risk ile açıkça hizalamaktır. Bir servisi yeterince güvenilir yapmaya çalışırız, ama olması gerekenden _daha fazla_ güvenilir değil. Yani, %99.99 availability target'ı koyduğumuzda, bunu aşmak istiyoruz, ama çok fazla değil: bu sisteme özellik ekleme, technical debt'i temizleme veya operational maliyetlerini azaltma fırsatlarını harcamak olur. Bir anlamda, availability target'ını hem minimum hem de maksimum olarak görüyoruz. Bu çerçevelemenin temel avantajı açık, düşünceli risk alma'nın kilidini açmasıdır.

# Servis Risk'ini Ölçmek

Google'da standart uygulama olarak, optimize etmek istediğimiz sistem özelliğini temsil edecek objective bir metrik belirlememiz sıklıkla en iyi hizmet eder. Bir target belirleyerek, mevcut performansımızı değerlendirebilir ve zaman içinde iyileştirmeleri veya kötüleşmeleri takip edebiliriz. Servis riski için, tüm potansiyel faktörleri tek bir metriğe nasıl indirgeceğimiz hemen belli değildir. Servis arızalarının user dissatisfaction, zarar veya güven kaybı; doğrudan veya dolaylı gelir kaybı; marka veya itibar etkisi; ve istenmeyen basın kapsamı dahil birçok potansiyel etkisi olabilir. Açıkça, bu faktörlerin bazıları ölçülmesi çok zordur. Bu problemi ele alınabilir ve çalıştırdığımız birçok sistem türü arasında tutarlı hale getirmek için _unplanned downtime_'a odaklanıyoruz.

Çoğu servis için risk toleransını temsil etmenin en basit yolu kabul edilebilir unplanned downtime seviyesi açısındandır. Unplanned downtime istenen _service availability_ seviyesiyle yakalanır, genellikle sağlamak istediğimiz "nine" sayısı açısından ifade edilir: %99.9, %99.99 veya %99.999 availability. Her ek nine %100 availability'ye doğru büyüklük sırası iyileştirmesine karşılık gelir. Serving sistemler için, bu metrik geleneksel olarak sistem uptime'ının oranına dayalı olarak hesaplanır (bkz. Time-based availability).

##### Time-based availability

**Time-based availability = Uptime / (Uptime + Downtime)**

Bu formülü bir yıllık süre boyunca kullanarak, belirli sayıda nine availability'ye ulaşmak için kabul edilebilir downtime dakikalarını hesaplayabiliriz. Örneğin, %99.99 availability target'ına sahip bir sistem yılda 52.56 dakikaya kadar down olabilir ve availability target'ı içinde kalabilir.

Google'da ise availability için time-based metrik genellikle anlamlı değildir çünkü global olarak dağıtılmış servislere bakıyoruz. Fault isolation yaklaşımımız, herhangi bir zamanda dünyanın herhangi bir yerinde belirli bir servis için en azından trafik alt kümesini serve ediyor olmamızı çok olası kılar (yani, her zaman en azından kısmen "ayakta"yız). Bu nedenle, uptime etrafındaki metrikler kullanmak yerine, availability'yi _request success rate_ açısından tanımlıyoruz. Aggregate availability bu yield-based metriğin rolling window üzerinde nasıl hesaplandığını gösterir (yani, bir günlük window üzerinde başarılı request'lerin oranı).

##### Aggregate availability

**Aggregate availability = Successful requests / Total requests**

Örneğin, günde 2.5M request serve eden ve %99.99 günlük availability target'ına sahip bir sistem 250'ye kadar error serve edebilir ve o gün için hala target'ını tutturabilir.

Tipik bir uygulamada, tüm request'ler eşit değildir: yeni kullanıcı kayıt request'inin başarısız olması background'da yeni email için polling yapan request'in başarısız olmasından farklıdır. Bununla birlikte birçok durumda, tüm request'ler üzerinden request success rate olarak hesaplanan availability, end-user perspektifinden görüldüğü şekliyle unplanned downtime'ın makul bir yaklaşımıdır.

Unplanned downtime'ı request success rate olarak quantify etmek, bu availability metriğini genellikle end user'lara doğrudan servis vermeyen sistemlerde kullanım için daha uygun hale getirir. Çoğu nonserving sistem (örneğin, batch, pipeline, storage ve transactional sistemler) başarılı ve başarısız work unit'lerinin iyi tanımlanmış bir kavramına sahiptir.

Örneğin, müşteri database'lerinden birinin içeriğini extract eden, transform eden ve daha fazla analiz sağlamak için data warehouse'a insert eden batch process periyodik olarak çalışacak şekilde ayarlanabilir. Başarılı ve başarısızla işlenmiş record'lar açısından tanımlanan request success rate kullanarak, batch sistemin sürekli çalışmamasına rağmen faydalı bir availability metriği hesaplayabiliriz.

Çoğunlukla, bir servis için quarterly availability target'ları belirleriz ve bu target'lara karşı performansımızı haftalık, hatta günlük bazda takip ederiz. Bu strateji, kaçınılmaz olarak ortaya çıkan anlamlı sapmaları arayarak, takip ederek ve düzelterek servisi yüksek seviye availability objective'ine göre yönetmemizi sağlar.

# Servislerin Risk Toleransı

Bir servisin risk toleransını belirlemenin anlamı nedir? Formal bir environmentda veya safety-critical sistemler durumunda, servislerin risk toleransı genellikle temel ürün veya servis tanımına doğrudan dahil edilir. Google'da, servislerin risk toleransı daha az açık şekilde tanımlanma eğilimindedir.

Bir servisin risk toleransını belirlemek için, SRE'ler product owner'larla birlikte business hedeflerini engineering yapabileceğimiz açık objective'lere dönüştürmek için çalışmalıdır. Bu durumda, ilgilendiğimiz business hedefleri sunulan servisin performansı ve güvenilirliği üzerinde doğrudan etkiye sahiptir. Pratikte, bu çeviri söylemesi yapmaktan daha kolaydır. Consumer servisler genellikle bir uygulama için business owner'ı olarak hareket eden product team'e sahipken, infrastructure servisler (örneğin, storage sistemleri veya genel amaçlı HTTP caching katmanı) için benzer product ownership yapısına sahip olmak alışılmadıktır.

## Consumer Servislerinin Risk Toleransını Belirleme

Consumer servislerimiz sıklıkla bir uygulama için business owner olarak hareket eden product team'e sahiptir. Örneğin, Search, Google Maps ve Google Docs'un her birinin kendi product manager'ları vardır. Bu product manager'lar kullanıcıları ve business'ı anlamak ve ürünü pazarda başarı için şekillendirmekle görevlidirler. Bir product team mevcut olduğunda, o team genellikle bir servisin güvenilirlik gereksinimlerini tartışmak için en iyi kaynaktır.

Servislerin risk toleransını değerlendirirken göz önünde bulundurulacak birçok faktör vardır:

* Hangi seviye availability gereklidir?
* Farklı failure türlerinin servis üzerinde farklı etkileri var mı?
* Servisi risk continuum'unda konumlandırmak için servis maliyetini nasıl kullanabiliriz?
* Hesaba katılması önemli olan diğer servis metrikleri nelerdir?

### Target availability seviyesi

Belirli bir Google servisi için target availability seviyesi genellikle sağladığı fonksiyona ve pazarda nasıl konumlandırıldığına bağlıdır. Göz önünde bulundurulacak konular şunları içerir:

* Kullanıcılar hangi seviye servisi bekleyecek?
* Bu servis doğrudan gelire bağlı mı (bizim gelirimiz veya müşterilerimizin geliri)?
* Bu ücretli bir servis mi, yoksa ücretsiz mi?
* Pazarda rakipler varsa, bu rakipler hangi seviye servisi sağlıyor?
* Bu servis consumer'lara mı yoksa enterprise'lara mı yönelik?

Google Apps for Work gereksinimlerini düşünün. Kullanıcılarının çoğunluğu büyük ve küçük enterprise kullanıcılardır. Bu enterprise'lar çalışanlarının günlük işlerini yapmalarını sağlayan araçlar sunmak için Google Apps for Work servislerine (örneğin, Gmail, Calendar, Drive, Docs) bağımlıdır. Başka bir deyişle, Google Apps for Work servisi için bir outage sadece Google için değil, aynı zamanda kritik olarak bize bağımlı olan tüm enterprise'lar için bir outage'dir. Tipik bir Google Apps for Work servisi için, %99.9 external quarterly availability target'ı belirleyebilir ve bu target'ı daha güçlü internal availability target'ı ve external target'ı karşılayamazsak cezalar öngören sözleşmeyle destekleyebiliriz.

YouTube karşıt bir düşünceler seti sunar. Google YouTube'u satın aldığında, website için uygun availability target'ını kararlaştırmak zorundaydık. 2006'da YouTube consumer'lara odaklanmıştı ve o zamanki Google'dan çok farklı business lifecycle aşamasındaydı. YouTube zaten harika bir ürüne sahipken, hala değişiyor ve hızla büyüyordu. Hızlı özellik geliştirme buna bağlı olarak daha önemli olduğu için YouTube için enterprise ürünlerimizden daha düşük availability target'ı belirledik.

### Failure türleri

Belirli bir servis için beklenen failure şekli başka bir önemli düşüncedir. Business'ımız servis downtime'ına ne kadar resilient? Servis için hangisi daha kötü: sürekli düşük failure oranı mı yoksa ara sıra tam site outage'ı mı? Her iki failure türü de aynı absolute error sayısıyla sonuçlanabilir, ama business üzerinde büyük ölçüde farklı etkiler yaratabilir.

Full ve partial outage'lar arasındaki farkın açıklayıcı örneği özel bilgi serve eden sistemlerde doğal olarak ortaya çıkar. Bir contact management uygulamasını ve profile resimlerinin render olamamasına neden olan intermittent failure'lar ile bir kullanıcının özel contact'larının başka bir kullanıcıya gösterilmesiyle sonuçlanan failure case arasındaki farkı düşünün. İlk durum açıkça kötü bir user experience'dır ve SRE'ler problemi hızla düzeltmek için çalışır. İkinci durumda ise, özel verileri açığa çıkarma riski temel kullanıcı güvenini önemli ölçüde sarsabilir. Sonuç olarak, ikinci durum için debugging ve potansiyel temizlik aşamasında servisi tamamen kapatmak uygun olacaktır.

Google'ın sunduğu servislerin diğer ucunda, maintenance window'ları sırasında düzenli outage'lara sahip olmak bazen kabul edilebilirdir. Birkaç yıl önce, Ads Frontend böyle bir servis olarak kullanılıyordu. Advertiser'lar ve website publisher'lar tarafından advertising campaign'lerini kurmak, konfigüre etmek, çalıştırmak ve monitör etmek için kullanılır. Bu işin çoğu normal business saatleri içinde gerçekleştiği için, maintenance window'ları şeklinde ara sıra, düzenli, planlanmış outage'ların kabul edilebilir olacağını belirledik ve bu planlanmış outage'ları unplanned downtime değil, planned downtime olarak saydık.

### Maliyet

Maliyet sıklıkla bir servis için uygun availability target'ını belirlemede kilit faktördür. Ads request başarıları ve başarısızlıklarının doğrudan kazanılan veya kaybedilen gelire çevrilebilmesi nedeniyle bu trade-off'u yapmak için özellikle iyi bir konumdadır. Her servis için availability target'ını belirlerken şöyle sorular sorarız:

* Bu sistemleri bir nine daha availability ile inşa edip işletsek, incremental gelir artışımız ne olurdu?
* Bu ek gelir o güvenilirlik seviyesine ulaşma maliyetini karşılıyor mu?

Bu trade-off denklemini daha somut hale getirmek için, her request'in eşit değere sahip olduğu örnek bir servis için şu cost/benefit'i düşünün:

* Önerilen availability target'ı iyileştirmesi: %99.9 → %99.99
* Önerilen availability artışı: %0.09
* Servis geliri: $1M
* İyileştirilmiş availability'nin değeri: $1M * 0.0009 = $900

Bu durumda, availability'yi bir nine iyileştirme maliyeti $900'dan azsa, yatırıma değer. Maliyet $900'dan fazlaysa, maliyetler öngörülen gelir artışını aşacaktır.

Güvenilirlik ve gelir arasında basit bir çeviri fonksiyonumuz olmadığında bu target'ları belirlemek daha zor olabilir. Faydalı bir strateji Internet'teki ISP'lerin background error rate'ini göz önünde bulundurmak olabilir. Failure'lar end-user perspektifinden ölçülüyorsa ve servis için error rate'ini background error rate'inin altına düşürmek mümkünse, bu error'lar belirli bir kullanıcının Internet bağlantısı için noise içinde kalacaktır. ISP'ler ve protokoller arasında önemli farklar olmasına rağmen (örneğin, TCP versus UDP, IPv4 versus IPv6), ISP'ler için tipik background error rate'ini %0.01 ve %1 arasında düşen olarak ölçmüşüzdür.

### Diğer servis metrikleri

Servislerin risk toleransını availability dışındaki metriklerle ilişkili olarak incelemek sıklıkla faydalıdır. Hangi metriklerin önemli ve hangilerinin önemli olmadığını anlamak, düşünceli risk almaya çalışırken bize serbestlik dereceleri sağlar.

Ads sistemlerimiz için servis latency açıklayıcı bir örnek sunar. Google ilk Web Search'ü başlattığında, servisin temel ayırt edici özelliklerinden biri hızdı. Search sonuçlarının yanında reklam gösteren AdWords'ü tanıttığımızda, sistemin temel gereksinimi reklamların search experience'ını yavaşlatmaması oldu. Bu gereksinim her AdWords sistem generasyonunda engineering hedeflerini yönlendirmiş ve invariant olarak görülmüştür.

Publisher'ların website'lerine ekledikleri JavaScript kodu isteklerine yanıt olarak contextual reklam serve eden Google'ın reklam sistemi AdSense, çok farklı bir latency hedefine sahiptir. AdSense için latency hedefi, contextual reklam eklerken third-party sayfanın render'ını yavaşlatmaktan kaçınmaktır. Spesifik latency target'ı, o halde, belirli bir publisher'ın sayfasının render olma hızına bağımlıdır. Bu, AdSense reklamlarının genellikle AdWords reklamlarından yüzlerce milisaniye daha yavaş serve edilebileceği anlamına gelir.

Bu daha gevşek serving latency gereksinimi, kullandığımız serving kaynaklarının miktarını ve lokasyonlarını belirlemede naive provisioning üzerinden önemli maliyet kazandıran birçok akıllı trade-off yapmamızı sağlamıştır. Başka bir deyişle, AdSense servisinin latency performansındaki ılımlı değişikliklere göreceli duyarsızlığı göz önüne alındığında, serving'i daha az geografik lokasyona konsolide edebilir, operational overhead'imizi azaltabiliriz.

## Infrastructure Servislerinin Risk Toleransını Belirleme

Infrastructure komponentlerini inşa etme ve çalıştırma gereksinimleri consumer ürünlerin gereksinimlerinden birkaç açıdan farklıdır. Temel bir fark, tanımı gereği, infrastructure komponentlerinin genellikle değişen ihtiyaçları olan birden fazla client'a sahip olmasıdır.

### Target availability seviyesi

Massive-scale distributed storage sistemi Bigtable'ı düşünün. Bazı consumer servisler bir user request'inin path'inde doğrudan Bigtable'dan veri serve eder. Bu tür servisler düşük latency ve yüksek güvenilirliğe ihtiyaç duyar. Diğer takımlar düzenli bazda offline analiz (örneğin, MapReduce) gerçekleştirmek için kullandıkları verilerin repository'si olarak Bigtable kullanır. Bu takımlar güvenilirlikten ziyade throughput'la ilgilenme eğilimindedir. Bu iki use case için risk toleransı oldukça farklıdır.

Her iki use case'in ihtiyaçlarını karşılamanın bir yaklaşımı tüm infrastructure servislerini ultra-güvenilir olacak şekilde engineer etmektir. Bu infrastructure servislerinin ayrıca büyük miktarda kaynağı aggregate etme eğiliminde olduğu gerçeği göz önüne alındığında, böyle bir yaklaşım genellikle pratikte çok pahalıdır.

### Failure türleri

Düşük latency kullanıcısı Bigtable'ın request queue'larının (neredeyse her zaman) boş olmasını ister, böylece sistem her outstanding request'i varışında hemen işleyebilir. (Gerçekten de, inefficient queuing sıklıkla yüksek tail latency'nin nedenidir.) Offline analiz ile ilgilenen kullanıcı sistem throughput'uyla daha çok ilgilenir, bu yüzden o kullanıcı request queue'larının asla boş olmamasını ister. Throughput için optimize etmek için, Bigtable sistemi bir sonraki request'ini beklerken hiç idle kalmamalıdır.

Gördüğünüz gibi, başarı ve başarısızlık bu kullanıcı setleri için antitetiktir. Düşük latency kullanıcısı için başarı, offline analiz ile ilgilenen kullanıcı için başarısızlıktır.

### Maliyet

Bu rekabet eden kısıtlamaları cost-effective şekilde tatmin etmenin bir yolu infrastructure'ı partition etmek ve birden fazla bağımsız servis seviyesinde sunmaktır. Bigtable örneğinde, iki tip cluster inşa edebiliriz: düşük latency cluster'ları ve throughput cluster'ları. Düşük latency cluster'ları düşük latency ve yüksek güvenilirliğe ihtiyaç duyan servisler tarafından operate edilmek ve kullanılmak için tasarlanır. Kısa queue uzunluklarını garanti etmek ve daha stringent client isolation gereksinimlerini tatmin etmek için, Bigtable sistemi azaltılmış contention ve artırılmış redundancy için önemli miktarda slack capacity ile provision edilebilir. Öte yandan, throughput cluster'ları latency üzerinden throughput'u optimize ederek çok sıcak ve daha az redundancy ile çalışacak şekilde provision edilebilir. Pratikte, bu relaxed ihtiyaçları düşük latency cluster maliyetinin belki %10-50'si kadar çok daha düşük maliyetle tatmin edebiliyoruz. Bigtable'ın massive scale'i göz önüne alındığında, bu maliyet tasarrufu çok hızla önemli hale gelir.

Infrastructure ile ilgili kilit strateji, client'ların sistemlerini inşa ederken doğru risk ve maliyet trade-off'larını yapmalarını sağlayacak şekilde açıkça sınırlandırılmış servis seviyelerine sahip servisler sunmaktır. Açıkça sınırlandırılmış servis seviyelerine sahip olmakla, infrastructure provider'ları belirli seviyede servis sağlamak için gereken maliyet farkını client'lara etkili şekilde externalize edebilir.

# Error Budget'ların Motivasyonu

**Yazan:** Mark Roth  
**Editör:** Carmela Quinito

Bu kitaptaki diğer bölümler product development takımları ve SRE takımları arasında nasıl gerilimler çıkabileceğini, genellikle farklı metriklerle değerlendirildikleri göz önüne alındığında tartışır. Product development performansı büyük ölçüde product velocity üzerinden değerlendirilir, bu da yeni kodu mümkün olduğunca hızlı push etme incentive'i yaratır. Bu arada, SRE performansı (şaşırtıcı olmayan şekilde) bir servisin güvenilirliğine dayalı olarak değerlendirilir, bu da yüksek değişim oranına karşı push back yapma incentive'ini içerir. İki takım arasındaki information asymmetry bu inherent gerilimleri daha da artırır. Product developer'lar kodlarını yazma ve release etmeye dahil olan zaman ve eforla ilgili daha fazla visibility'ye sahipken, SRE'ler servisin güvenilirliği (ve genel olarak production'ın durumu) konusunda daha fazla visibility'ye sahiptir.

Bu gerilimler sıklıkla kendilerini engineering pratiklerine konulması gereken efor seviyesi hakkında farklı görüşlerde yansıtır:

**Software fault tolerance**
Software'ı beklenmeyen olaylara karşı ne kadar sertleştiriyoruz? Çok az, kırılgan, kullanılamaz bir ürün elde ederiz. Çok fazla, kimsenin kullanmak istemediği (ama çok stabil çalışan) bir ürün elde ederiz.

**Testing**
Yine, yetersiz test utanç verici outage'lar, privacy veri sızıntıları veya diğer basın değeri taşıyan olaylarla sonuçlanır. Çok fazla test, pazarınızı kaybedebilirsiniz.

**Push frequency**
Her push risklidir. Bu riski azaltma üzerinde ne kadar çalışmalıyız, diğer işler yapmaya karşı?

**Canary duration ve boyutu**
Yeni release'i tipik workload'ın küçük bir alt kümesinde test etmek, genellikle _canarying_ adı verilen bir best practice'tir. Ne kadar bekliyoruz ve canary ne kadar büyük?

Genellikle, önceden var olan takımlar risk/efor sınırının nerede durduğuna dair aralarında bir tür informal denge kurmuştur. Ne yazık ki, bu dengenin optimal olduğunu nadiren kanıtlayabilirsiniz; dahil olan engineer'ların negotiation becerilerinin bir fonksiyonu olmaktan ziyade. Bu tür kararlar siyaset, korku veya umutla yönlendirilmemelidir.

## Error Budget'ınızı Oluşturmak

Bu kararları objective veriye dayandırmak için, iki takım birlikte servisin service level objective'ına (SLO) dayalı quarterly error budget tanımlar. Error budget servisin tek bir quarter içinde ne kadar güvenilmez olmasına izin verildiğini belirleyen açık, objective metrik sağlar. Bu metrik, ne kadar riske izin verileceğine karar verirken SRE'ler ve product developer'lar arasındaki müzakerelerdeki siyaseti ortadan kaldırır.

Pratiğimiz şöyledir:

* Product Management servisin quarter başına ne kadar uptime'a sahip olması gerektiğine dair expectation belirleyen bir SLO tanımlar.
* Gerçek uptime neutral üçüncü taraf tarafından ölçülür: monitoring sistemimiz.
* Bu iki sayı arasındaki fark quarter için kalan "güvenilmezlik" budget'ıdır.
* Ölçülen uptime SLO'nun üstünde olduğu sürece—başka bir deyişle, kalan error budget'ı olduğu sürece—yeni release'ler push edilebilir.

Örneğin, bir servisin SLO'sunun quarter başına tüm sorguların %99.999'unu başarıyla serve etmek olduğunu hayal edin. Bu, servisin error budget'ının belirli bir quarter için %0.001 failure rate'i olduğu anlamına gelir. Bir problem quarter için beklenen sorguların %0.0002'sinin başarısız olmasına neden olursa, problem servisin quarterly error budget'ının %20'sini harcar.

## Faydalar

Error budget'ının temel faydası hem product development hem de SRE'nin inovasyon ve güvenilirlik arasında doğru dengeyi bulmaya odaklanmasını sağlayan ortak incentive sağlamasıdır.

Birçok ürün release velocity'yi yönetmek için bu control loop'u kullanır: sistemin SLO'ları karşılandığı sürece, release'ler devam edebilir. SLO violation'ları error budget'ını harcayacak kadar sık meydana gelirse, sistem daha resilient yapmak, performansını artırmak vb. için ek kaynaklar yatırılırken release'ler geçici olarak durdurulur.

Örneğin, product development test yapmayı azaltmak veya push velocity'yi artırmak istiyorsa ve SRE direniş gösteriyorsa, error budget kararı yönlendirir. Budget büyük olduğunda, product developer'lar daha fazla risk alabilir. Budget neredeyse tükendiğinde, product developer'lar kendileri daha fazla test veya daha yavaş push velocity için bastırır, çünkü budget'ı tüketip launch'larını durdurmak istemezler. Aslında, product development takımı kendi kendini denetler hale gelir. Budget'ı bilirler ve kendi risklerini yönetebilirler. (Tabii ki, bu sonuç SRE takımının SLO bozulursa launch'ları gerçekten durdurma yetkisine sahip olmasına dayanır.)

Network outage'ı veya datacenter arızası ölçülen SLO'yu azaltırsa ne olur? Bu tür olaylar da error budget'ını yer. Sonuç olarak, quarter'ın geri kalanında yeni push'ların sayısı azaltılabilir. Tüm takım bu azalmayı destekler çünkü herkes uptime sorumluluğunu paylaşır.

Budget ayrıca aşırı yüksek güvenilirlik target'larının hem esneklik hem de yavaş inovasyon açısından maliyetlerini vurgulamaya yardımcı olur. Takım yeni özellikler launch etmekte zorlanıyorsa, inovasyonu artırmak için SLO'yu gevşetmeyi (böylece error budget'ını artırmayı) seçebilir.

##### Temel Çıkarımlar

* Servis güvenilirliğini yönetmek büyük ölçüde risk'i yönetmekle ilgilidir ve risk'i yönetmek maliyetli olabilir.
* %100 muhtemelen asla doğru güvenilirlik target'ı değildir: sadece ulaşılması imkansız olmakla kalmaz, genellikle bir servisin kullanıcılarının istediği veya fark ettiğinden daha fazla güvenilirliktir. Servisin profilini business'ın almaya istekli olduğu riskle eşleştirin.
* Error budget incentive'leri hizalar ve SRE ile product development arasında ortak sahiplenmeyi vurgular. Error budget'lar release oranına karar vermeyi kolaylaştırır ve stakeholder'larla outage tartışmalarını etkili şekilde etkisizleştirir, ve birden fazla takımın rancor olmadan production risk'i hakkında aynı sonuca varmasını sağlar.