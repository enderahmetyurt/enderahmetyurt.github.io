---
layout: post
title: 'Ruby’de Abstract Factory Tasarım Deseni'
date: 2025-06-22 18:18 +0300
categories: [Development]
tags: [ruby,design-patterns,türkçe]
image:
  path: https://images.unsplash.com/photo-1578776349090-de61da00ff1a?fm=jpg&q=60&w=3000&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

Merhaba 👋

Uygulamalar büyüdükçe, birbiriyle uyumlu şekilde çalışması gereken nesne gruplarıyla karşılaşırız. Farklı veritabanı motorlarına göre konfigürasyon nesneleri, çoklu tema desteği olan kullanıcı arayüzleri, ya da bölgelere özel ödeme sistemleri… Bu gibi durumlarda yalnızca bir nesneyi değil, bir **ailenin tüm üyelerini** üretmemiz gerekir — hem de birbirleriyle tutarlı bir şekilde.

**Abstract Factory** deseni, bu tür senaryolarda hayat kurtarır.

Bu desen sayesinde, ilişkili nesneleri birbirine sıkı sıkıya bağlamadan üretebiliriz. Yani “hangi nesneyi kullanacağımızı” bilen değil, “hangi aileden nesne kullanacağımızı” bilen bir sistem tasarlarız.

Ruby’nin esnek yapısı nedeniyle bu desen ilk bakışta gereksiz soyutlama gibi görünebilir. Ancak özellikle **çoklu yapılandırmalar**, **değişebilir dış servisler** ve **arayüz varyasyonları** içeren projelerde bu desenin avantajı açıkça ortaya çıkar.

Bu yazıda, Abstract Factory desenini Ruby ile adım adım inceleyecek; önce sade bir örnek üzerinden temelleri anlayacak, ardından gerçek hayattaki kullanım senaryolarına göz atacağız.

## Örnek: Temaya Göre UI Bileşenleri Üretmek
Bir uygulamada kullanıcıya koyu (dark) veya açık (light) tema seçme imkanı sunmak istiyoruz. Bu temaya göre butonlar, giriş alanları (input), başlıklar gibi bileşenler farklı şekilde görünmeli. Ve tabii ki bu bileşenlerin birbiriyle tutarlı olması gerekiyor.

Abstract Factory yaklaşımıyla şöyle ilerleyeceğiz:

- Her tema için bir "fabrika" tanımlarız.
- Bu fabrikalar, kendi stillerine uygun bileşenleri üretir.
- Uygulamanın geri kalanı, bu temaya özgü nesnelerin nasıl oluşturulduğuyla ilgilenmez.

```ruby
# Abstract factory
class UIFactory
  def create_button; raise NotImplementedError; end
  def create_input;  raise NotImplementedError; end
end

# Concrete factories
class DarkThemeFactory < UIFactory
  def create_button
    DarkButton.new
  end

  def create_input
    DarkInput.new
  end
end

class LightThemeFactory < UIFactory
  def create_button
    LightButton.new
  end

  def create_input
    LightInput.new
  end
end

# Product families
class DarkButton
  def render
    "[Dark] Buton"
  end
end

class LightButton
  def render
    "[Light] Buton"
  end
end

class DarkInput
  def render
    "[Dark] Giriş Alanı"
  end
end

class LightInput
  def render
    "[Light] Giriş Alanı"
  end
end

# Client
def render_ui(factory)
  button = factory.create_button
  input  = factory.create_input

  puts button.render
  puts input.render
end

# Uygulama başlangıcında tema seçilir
theme = ENV['UI_THEME'] == "dark" ? DarkThemeFactory.new : LightThemeFactory.new
render_ui(theme)
```

### 🧠 Ne oldu burada?
- `UIFactory` soyut bir üretim arayüzü sağladı.
- Her tema, kendi `Button` ve `Input` sınıflarını üretmekle sorumlu oldu.
- `render_ui` fonksiyonu, nesnelerin *kim* olduğunu bilmeden onları kullanabildi.

Bu yapı sayesinde yeni bir tema eklemek istediğimizde, sadece yeni bir fabrika ve bileşen ailesi tanımlarız. Uygulamanın diğer kısımlarına dokunmamıza gerek kalmaz.

## 🌍 Gerçek Dünyada Abstract Factory Ne İşe Yarar?
Abstract Factory deseni özellikle şu durumlarda parlamaya başlar:

### 1. Üçüncü Parti Entegrasyonlar Arasında Geçiş Yapmak
> Diyelim ki uygulaman birden fazla ödeme sağlayıcısını (örneğin: Stripe, Iyzico, PayPal) destekliyor.
> Her birinin “form alanları”, “ödeme işlem mantığı”, “webhook yapısı” farklı.

Bu durumda:

- `PaymentGatewayFactory` gibi bir soyut sınıf tanımlarsın.
- Her sağlayıcı için birer `ConcreteFactory` yazarsın (örneğin `StripeFactory`, `IyzicoFactory`).
- Bu fabrikalar hem `PaymentForm`, hem `PaymentProcessor`, hem `WebhookHandler` gibi ilgili sınıfları üretir.

Kazanım:

- Ödeme sağlayıcısını değiştirmek bir `Factory` değişikliğine indirgenir.
- Diğer kodlar Stripe mı Iyzico mu olduğunu bilmez.

### 2. Uluslararasılaştırılmış (i18n) UI/İçerik Yönetimi
> Kullanıcının bulunduğu bölgeye göre farklı dilde ve biçimde arayüz öğeleri üretmek.

Örnek:

- `TRLocaleFactory`, `ENLocaleFactory`, `DELocaleFactory`
- Bu fabrikalar “greeting mesajı”, “tarih formatı”, “para birimi biçimi” gibi nesneleri üretir.

### 3. Farklı “Domain”e Özgü Logic’i Ayrıştırmak
> Örneğin: e-Fatura sağlayıcıları (Paraşüt, Mikro, Logo) her biri benzer işlemleri yapar ama protokol ve mapping farklıdır.

Burada:

- `EFaturaFactory` soyut sınıfı ile her sağlayıcının `EFaturaSender`, `DocumentFormatter`, `ResultParser` gibi bileşenleri oluşturulur.

### 4. Test Ortamlarında Mock Nesneleri Kullanmak
> Production’da gerçek nesneler, testte sahte/mock nesneler üretmek istiyorsan…

- `RealServiceFactory` ve `FakeServiceFactory` sınıfları aynı arayüzden “uyumlu nesneler ailesi” oluşturur.
- Böylece testte tek bir factory değiştirerek tüm bağlı sınıfların test versiyonlarını kullanabilirsin

## 🤝 Özetle
Abstract Factory deseni;

- Uyumlu nesnelerin birlikte üretilmesi gerektiği,
- Kodun bu nesnelerin *hangi sınıfa ait olduğunu bilmemesi* gerektiği,
- Her bir "ortam", "tema", "sağlayıcı" için aynı API'ye sahip ama farklı içeriğe sahip nesne ailelerinin gerektiği durumlarda kullanılır.

Bu desen “gereksiz soyutlama” gibi görünse de, doğru yerde kullanıldığında sistemin **esnekliğini artırır, bağımlılıkları azaltır ve test edilebilirliği güçlendirir.**

## 🧭 Ruby’de Abstract Factory Ne Zaman Gerekli, Ne Zaman Aşırıya Kaçar?
Ruby gibi esnek ve duck typing’e dayalı bir dilde, bazı desenler gereğinden fazla soyutlama yaratabilir. Abstract Factory de bu riskli desenlerden biridir. Peki ne zaman kullanmalı?

### ✅ Kullan:

- Birbirine bağlı birden fazla nesneyi birlikte üretmen gerekiyorsa.
- Yarın sağlayıcı/tema/lokalizasyon değiştiğinde sadece bir factory dosyasıyla sistemi adapte etmek istiyorsan.
- Test ortamında tüm bağımlı servisleri tek factory üzerinden “mock” versiyonlarla değiştirmek istiyorsan.

### ❌ Kullanma:

- Sadece **tek bir nesne türü** yaratacaksan.
- Nesneler arası bir **bağlılık yoksa** (birbirine uyumlu olmak zorunda değillerse).
- Her factory’nin içinde 1–2 satırlık kod varsa ve bu kodlar değişmeyecekse → bu durumda `Factory Method` daha uygun olabilir.

### 🎯 Ruby’de Alternatifler:

- Basit `Hash` veya `case/when` yapıları ile dependency seçmek.
- Dependency Injection (örneğin `MyService.new(client: StripeClient.new)`).
- Duck typing ile arayüz kontrolü (`respond_to?`, `public_send` vs).

## ✍️ Görüşler

Abstract Factory, Ruby’de nadiren ihtiyaç duyulan ama doğru yerde kullanıldığında ciddi esneklik sağlayan bir desen. Büyük projelerde, entegre çalışan nesne ailelerinin çeşitlenmeye başladığı noktada devreye alınması sistemin sürdürülebilirliğini artırır.

Kodun geleceğe hazır olmasını istiyorsan, bazen bugünkü sadelikten ödün vermek gerekir. Ama unutmadan: her soyutlama, bir borçtur. Geri ödeyebileceksen yaz.

❤️