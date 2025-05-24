# AI.md – Yapay Zeka Kullanım Raporu

## 1. Kullanılan Araçlar

- *ChatGPT (https://chat.openai.com)*  
  Kod yazımı, oyun mekaniği planlama, README oluşturma, teknik sorunları çözme gibi tüm geliştirme sürecinde destek alındı.

- *ElevenLabs (https://elevenlabs.io)*  
  Oyun tanıtım videosu için anlatım metnini Türkçe olarak seslendirme amacıyla kullanıldı.

---

## 2. ChatGPT Kullanım Detayları

### Prompt 1:
*Kullanıcı:*  
Sesi videonun başına nasıl hizalayabilirim?

*ChatGPT:*  
CapCut, Shotcut veya Canva gibi programlarda sesi timeline üzerinde videonun başına sürükleyerek hizalayabilir ve “Export” butonuyla dışa aktarabilirsin.

---

### Prompt 2:
*Kullanıcı:*  
Oyuna başla butonuna ve tekrar dene butonuna tıklanınca bir tık sesi çıksın istiyorum.

*ChatGPT:*  
HTML’ye <audio> etiketiyle ses eklenmeli. JavaScript içinde ilgili butonlara onclick fonksiyonu ile document.getElementById("tiklamaSesi").play(); kodu yazılmalı.

---

### Prompt 3:
*Kullanıcı:*  
Readme dosyasına resim nasıl eklenir?

*ChatGPT:*  
Resmi projenin içine koyduktan sonra şu şekilde Markdown koduyla eklenebilir:  
![Oyun Görseli](gorseller/oyun-ekrani.png)

---

### Prompt 4:
*Kullanıcı:*  
Düşmanla koruyucu gezegen çarpışacağı zaman oyun donuyor sebebi ne olabilir?

*ChatGPT:*  
Tamam, bu donma (freeze) problemi muhtemelen çarpışma sesini tekrar tekrar oynatırken tarayıcının takılmasıyla ilgili olabilir. Genellikle bu, audio.play() çağrısı henüz bitmeden tekrar oynatılmaya çalışıldığında oluşur.

Çözüm: Her çarpışmada aynı ses nesnesini tekrar kullanmak yerine, yeni bir ses nesnesi yaratmak daha güvenli bir yöntemdir. Ama bunu da çok sık yaparsan performans düşebilir. En ideal yöntem:

1. Sesin her çarpışmada düzgün çalınabilmesi için cloneNode() kullan:
Kodun şu şekilde olsun:

function carpismaSesiCal() {

    const carpma = carpismaSesi.cloneNode(); // Aynı sesten yeni bir kopya
    
    carpma.play();
}

3. Eğer yukarıdaki zaten buysa ama hala donuyorsa:
Donmaya neden olan yer şurası olabilir:
Çarpışma kontrolü içinde carpismaSesiCal() çok sık ve art arda tetikleniyorsa (örneğin her frame’de çarpışma devam ediyorsa), o zaman sürekli ses oluşturuluyor demektir.
Bunu engellemek için:

let carpistiMi = false;

function carpismalariKontrolEt() {

    for (let i = dusmanlar.length - 1; i >= 0; i--) {
        const d = dusmanlar[i];
        const dx = d.x - koruyucuX;
        const dy = d.y - koruyucuY;
        const mesafe = Math.sqrt(dx * dx + dy * dy);

        if (mesafe < d.yaricap + koruyucuYaricap) {
            if (!carpistiMi) {
                carpismaSesiCal();
                carpistiMi = true;

                // Can azalt, düşmanı sil vs.
                can--;
                dusmanlar.splice(i, 1);

                setTimeout(() => {
                    carpistiMi = false; // 0.5 saniye sonra tekrar çalabilir
                }, 500);
            }
        }
    }
}

Bu sayede her çarpışmadan sonra 0.5 saniyelik bir “bekleme süresi” olur ve oyun donmaz.

---

### Prompt 5:
*Kullanıcı:*  
Düşmanlar oyunu başlat butonuna tıklamadan geliyor ve oyun başlatmadan bitiyor, nasıl düzeltebilirim?

*ChatGPT:*  
O zaman bu şu demek: setInterval oyun başlamadan önce bir yerlerde çağrılıyor. Yani büyük ihtimalle oyunBaslat() fonksiyonu dışında bir yerde dusmanOlustur() çağrılıyor ya da sayfa yüklenince oyunBaslat() otomatik çalışıyor.


Kontrol 1: Sayfa açıldığında oyunBaslat() çağrılıyor mu?
HTML dosyanda veya script'in en altında şunu arat:
oyunBaslat();
Eğer böyle bir satır varsa hemen sil ya da yorum satırı yap:
// oyunBaslat(); // BUNU SİL

Kontrol 2: dusmanOlustur() başka nerede çağrılıyor?
Kodlarında şunu ara:
dusmanOlustur();
Sadece şu satırda olması gerekiyor:

setInterval(() => {

    if (!oyunBitti) dusmanOlustur();
    
}, 1000);

Başka bir yerde (özellikle sayfa açılırken, window.onload gibi şeylerde) varsa, onu da sil.

Ekstra Kontrol: oyun hemen çiziliyor olabilir
Eğer şu satırı da en başta çağırıyorsan:
ciz();
Bu da düşmanları çalıştırabilir. Çünkü ciz() fonksiyonun içinde dusmanCizGuncelle() varsa, o da düşmanları hareket ettirir.

Ne Yap?
Kodlarının en altına bak, özellikle şunları kontrol et:
oyunBaslat() çağrılıyor mu?
ciz() çağrılıyor mu?
dusmanOlustur() doğrudan çağrılıyor mu?
Bu satırlardan biri varsa ve butondan önce çalışıyorsa, işte düşmanlar o yüzden erkenden geliyor.

---
### Prompt 6:
*Kullanıcı:*  
Arka plan sesi eklememe rağmen arka plan müziğim çalmıyor.

*ChatGPT:*  
Olası Sorunlar ve Çözümler:

1. Tarayıcı kısıtlaması:
Modern tarayıcılar (özellikle Chrome) otomatik olarak ses çalmaya izin vermez kullanıcı etkileşimi olmadan. Sen oyunBaslat() fonksiyonunu bir butona tıklamayla mı çağırıyorsun? Eğer bu fonksiyon bir tıklamayla tetikleniyorsa sorun yok. Ama otomatik çalışıyorsa, ses çalmaz.

Çözüm: arkaPlanMuzik.play() çağrısını mutlaka bir kullanıcı etkileşiminin (örneğin onclick) içinden yapmalısın. Mesela:

<button onclick="oyunBaslat()">Oyunu Başlat</button>


2. Aynı sesi birden fazla kez play etme:
Eğer kullanıcı oyunu birkaç kez başlatırsa arkaPlanMuzik.play() tekrar çağrılır. Bu bazen hata verir. O yüzden eklemeyi düşünebilirsin:

function oyunBaslat() {

    if (arkaPlanMuzik.paused) {
    
        arkaPlanMuzik.play();
    }
    
    setInterval(() => {
    
        if (!oyunBitti) dusmanOlustur();
        
    }, 1500);
    
    ciz();
}


3. Dosya yolu hatalı olabilir:
bgmusic.mp3 dosyası gerçekten aynı klasörde mi? Eğer /audio/ klasöründeyse, yol şöyle olmalı: "audio/bgmusic.mp3"

Özetle:

arkaPlanMuzik.play() mutlaka kullanıcı etkileşiminden sonra çağrılmalı.

Değişken adı hatasını düzelttin mi emin ol.

Dosya yolunu kontrol et.



