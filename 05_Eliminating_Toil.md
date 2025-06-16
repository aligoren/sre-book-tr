# Bölüm 5 - Toil'i Ortadan Kaldırmak

**Yazan:** Vivek Rau  
**Editör:** Betsy Beyer

Eğer bir insan bir operatoru çalıştırıyorsa, o zaman o operatör bozulmuştur.

> Carla Geisser, Google SRE

SRE'nin temel prensiplerinden biri, engineering işine odaklanmaktır. Engineering işi, uzun vadeli değer üreten, servisin ölçeklenmesini sağlayan ve güvenilirliğini artıran çalışmalardır. Ancak SRE'lerin zamanının önemli bir kısmı _toil_ adı verilen işlere harcanır.

## Toil Nedir?

Toil, production servislerini çalıştırmakla ilgili olan ancak aşağıdaki özelliklere sahip işlerdir:

**Manuel**
Otomatikleştirilebilir olmasına rağmen manuel olarak gerçekleştirilen işler.

**Tekrarlayan**
Aynı işi tekrar tekrar yapmak.

**Otomatikleştirilebilir**
Eğer bir insan şu anda yaptığı işi bir makine de yapabiliyorsa, o iş toil'dir.

**Taktiksel**
Interrupt-driven ve reactive, stratejik değil.

**Değer katmayan**
Servis zamanla aynı durumda kalır. Hiçbir kalıcı iyileştirme gerçekleşmez.

**O(n) ile büyür**
Servis büyüdükçe toil de doğrusal olarak büyür.

Eğer yukarıdaki özelliklerden beş veya altısına sahipse, muhtemelen toil'dir. Eğer sadece birkaçına sahipse, muhtemelen toil değildir. Örneğin, bir outage sırasında troubleshooting yapmak manuel ve interrupt-driven'dır, ancak genellikle tekrarlayan değildir ve kesinlikle değer katar.

### Toil Her Zaman Kötü Değildir

Toil'in her zaman kötü olduğunu söylemiyoruz. Herkesin biraz toil'e ihtiyacı vardır; bu, predictable ve manipülatif olabilir ve bazı insanlar için meditasyon benzeri bir kalite taşıyabilir. Toil, yeni takım üyelerinin production'a aşina olmalarına yardımcı olabilir. Ancak toil'in çok fazla olması kesinlikle kötüdür ve SRE organizasyonumuzun %50 toil hedefi bu gerçeği yansıtır.

### Toil Örnekleri

Tipik SRE toil kategorileri şunları içerir:

* Interrupt'lar (pager'lar dışında)
* On-call response'lar
* Manuel release'ler ve rollout'lar
* Capacity planning ve provisioning
* Değişiklik yönetimi
* Monitoring kurulumu
* Consulting
* Feature request'ler

## Neden Toil'den Kaçınmalıyız?

### Mühendislik Zamanı Azalır

Toil, engineering projelerine harcanabilecek zamanı tüketir. Engineering projeleri, toil'i azaltmak, servis güvenilirliğini artırmak veya performansı iyileştirmek için tasarlanmış projelerdir. Daha az engineering zamanı, daha az gelişme anlamına gelir.

### Kariyerde İlerleme Yavaşlar

Toil'de çok zaman harcayan mühendisler, kariyerlerinde daha yavaş ilerlerler. Google'da, mühendislik işi kariyerde ilerleme için gereklidir. Toil, mühendislerin yeni beceriler öğrenmesini ve karmaşık projeler üzerinde çalışmasını engeller.

### Moral Düşer

İnsanlar genellikle tekrarlayan, manuel işlerden sıkılırlar. Toil, mühendislerin motivasyonunu düşürür ve işten ayrılma oranını artırır.

### İlerleme Yavaşlar

Toil, takımların yeni özellikler geliştirmesini ve mevcut sistemleri iyileştirmesini yavaşlatır. Bu, ürünün rekabet gücünü azaltır.

### Precedent Yaratır

Toil kabul edilirse, daha fazla toil yaratılır. İnsanlar, mevcut toil'i görerek yeni toil yaratmanın kabul edilebilir olduğunu düşünürler.

### Attrition Yaratır

Toil, yetenekli mühendislerin şirketten ayrılmasına neden olur. Bu, takımın bilgi birikimini azaltır ve yeni mühendis işe alma maliyetlerini artırır.

### Breach of Faith

SRE'ler, engineering işi yapacakları beklentisiyle işe alınırlar. Çok fazla toil, bu beklentiyi karşılamaz ve güven kaybına neden olur.

## Toil'i Ölçmek

Toil'i ölçmek, onu azaltmak için kritiktir. Google'da, SRE takımları zamanlarının ne kadarını toil'e harcadıklarını düzenli olarak ölçerler.

### Toil Ölçüm Yöntemleri

**Zaman takibi**
Mühendisler, zamanlarını nasıl harcadıklarını kaydederler.

**Ticket analizi**
Gelen ticket'ların türleri ve sayıları analiz edilir.

**On-call analizi**
On-call sırasında harcanan zamanın türleri incelenir.

**Anketler**
Takım üyeleri, toil algıları hakkında anket doldururlar.

## Toil'i Ortadan Kaldırma Stratejileri

### Otomatikleştirme

En etkili toil azaltma yöntemi otomatikleştirmedir. Manuel işlemler, script'ler, araçlar veya sistemler aracılığıyla otomatikleştirilebilir.

**Otomatikleştirme Örnekleri:**
* Deployment pipeline'ları
* Monitoring ve alerting sistemleri
* Capacity planning araçları
* Self-service araçları

### Self-Service

Kullanıcıların kendi ihtiyaçlarını karşılayabilecekleri araçlar sağlamak, SRE'lerin interrupt'larını azaltır.

**Self-Service Örnekleri:**
* Developer'ların kendi deployment'larını yapabilmesi
* Kullanıcıların kendi hesap ayarlarını değiştirebilmesi
* Otomatik capacity scaling

### Süreç İyileştirme

Mevcut süreçleri gözden geçirmek ve iyileştirmek, toil'i azaltabilir.

**Süreç İyileştirme Örnekleri:**
* Gereksiz adımları kaldırmak
* Paralel işleme geçmek
* Batch işleme kullanmak

### Araç Geliştirme

Özel araçlar geliştirmek, tekrarlayan işleri azaltabilir.

**Araç Geliştirme Örnekleri:**
* Configuration management araçları
* Monitoring dashboard'ları
* Automated testing araçları

## Toil'i Ortadan Kaldırmanın Faydaları

### Mühendislik Zamanı Artar

Toil azaldıkça, mühendisler daha fazla zamanı engineering projelerine ayırabilirler.

### Güvenilirlik Artar

Engineering projeleri genellikle sistem güvenilirliğini artırır.

### Performans İyileşir

Otomatikleştirme ve araçlar, manuel işlemlerden daha hızlı ve tutarlıdır.

### Ölçeklenebilirlik Artar

Otomatikleştirilmiş sistemler, manuel süreçlerden daha iyi ölçeklenir.

### Moral Artar

Mühendisler, daha ilginç ve değerli işler üzerinde çalışabilirler.

## Toil Azaltma Projeleri

### Proje Seçimi

Toil azaltma projelerini seçerken şu faktörleri göz önünde bulundurun:

**Etki**
Proje ne kadar toil azaltacak?

**Çaba**
Proje ne kadar zaman ve kaynak gerektirecek?

**Risk**
Proje ne kadar riskli?

**Sürdürülebilirlik**
Çözüm uzun vadede sürdürülebilir mi?

### Proje Yürütme

Toil azaltma projelerini yürütürken:

1. **Mevcut durumu ölçün**
2. **Hedefleri belirleyin**
3. **Çözümü tasarlayın**
4. **Çözümü uygulayın**
5. **Sonuçları ölçün**
6. **İyileştirmeleri yapın**

## Sonuç

Toil, SRE'lerin kaçınması gereken ancak tamamen ortadan kaldırılamayan bir gerçekliktir. Amaç, toil'i makul seviyelerde tutmak ve sürekli olarak azaltmaya çalışmaktır. Bu, mühendislerin daha değerli işler üzerinde çalışmasını sağlar ve organizasyonun genel verimliliğini artırır.

Toil'i ortadan kaldırmak, sadece bireysel mühendislerin değil, tüm organizasyonun faydasınadır. Daha az toil, daha fazla innovation, daha iyi güvenilirlik ve daha mutlu mühendisler anlamına gelir.

SRE takımları, toil'i düzenli olarak ölçmeli, azaltma stratejileri geliştirmeli ve bu stratejileri sistematik olarak uygulamalıdır. Bu yaklaşım, SRE'nin temel prensiplerinden biri olan engineering odaklı çalışma kültürünü destekler. 