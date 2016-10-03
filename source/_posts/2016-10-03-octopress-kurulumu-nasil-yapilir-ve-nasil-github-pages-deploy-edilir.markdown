---
layout: post
title: "Octopress Kurulumu Nasıl Yapılır ve Nasıl Github Pages Deploy Edilir?"
date: 2016-10-03 10:32:01 +0300
comments: true
categories: [octopress, github, github pages]
---
Octopress statik bir site yönetimi sağlayan blog framework üdür. Terminalden yönettiğimiz blogumuzu Github pages deploy ederek aktif hale getirceğiz yazımda. Öncelikle "Octopress" i klonlamamız gerekiyor.

```bash
$ git clone git://github.com/imathis/octopress.git octopress
$ cd octopress
```
<!-- more -->

**Rake install**

Sırada ise default tema ve postlarımız için gerekli olan klasörleri bize sağlayacak olan kodu çalıştıracağız.

```bash
$ rake install
```

Eğer aşağıdaki gibi bir hata alırsak;

```bash
rake aborted!
Gem::LoadError: ...
```

"rake install" örneğindeki gibi "rake"  yazarak kullanacağımız bütün işlemlerin başına "bundle exec" yazmamız gerekecektir.

```bash
$ bundle exec rake install
```

**İlk Postumuzu oluşturalım**

Post oluşturmak için aşağıdaki kodu yazıyoruz.

```bash
$ bundle exec rake new_post
Enter a title for your post: Deneme 
```

Postumuzun adı için birşeyler yazıyoruz ve ilk postumuzu oluşturmuş oluyoruz. Ben "Deneme" adında bir post oluşturdum.

Bu ilk postumuz source klasörünün altında bulunan _posts klasörünün içinde bulunuyor ve [Markdown](https://daringfireball.net/projects/markdown/syntax "") syntax ile yazılması gerekiyor.

```bash
$ cd source/_posts 
```

yazarak postlarımıza ulaşıyoruz. İster subl, atom, rubymine vb. araçlarla açarak yönetebiliriz postumuzu.

```bash
---
layout: post
title: "Deneme"
date: 2016-10-03 09:26:18 +0300
comments: true
categories: 
---
```
Karşımıza bu şekilde başlayan bir sayfa açılacaktır. Bunun altına Markdown syntax ile yazdığımız yazımızı ekleyerek paylaşabiliyor olacağız.

Kategori aşamasını da aşağıda şekilde yazarak oluşturabiliyoruz.

```bash
categories: [octopress, github, github pages]
```

**Sitemizi "Generate" Edelim**

Yaptığımız değişikliklerden sonra *generate* komutunu çalıştırmalıyız ki değişiklikler oluşturulsun.

```bash
$ bundle exec rake generate
## Generating Site with Jekyll
directory source/stylesheets
    write source/stylesheets/screen.css
Configuration file: /home/esref/Documents/octopress/_config.yml
            Source: source
       Destination: public
      Generating... 
                    done.
```

**Sitemizi Önizleme Şeklinde Görüntüleyelim**

Oluşturduğumuz sitemizi local olarak görüntüleyebileceğiz. 

```bash
$bundle exec rake preview
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
Configuration file: /home/esref/Documents/octopress/_config.yml
>>> Compass is watching for changes. Press Ctrl-C to Stop.
            Source: source
       Destination: public
      Generating... 
[2016-10-03 09:45:32] INFO  WEBrick 1.3.1
[2016-10-03 09:45:32] INFO  ruby 2.3.0 (2015-12-25) [x86_64-linux]
[2016-10-03 09:45:32] INFO  WEBrick::HTTPServer#start: pid=20926 port=4000
                    done.
 Auto-regeneration: enabled for 'source'
    write public/stylesheets/screen.css
```

[http://localhost:4000/](http://localhost:4000/ "") sayfasında sitemizi görütüleyebiliriz. Sitemiz aşağıdaki şekildi görüntülene bilecektir. 

![Örnek Resim](http://i.hizliresim.com/yV9Eoy.png)

**Octopress'in Yapılandırılması(Configuring)**

Site adı, açıklama vb. bir çok değişikliği ve ayarları yapabileceğiniz dosya "_config.yml" dosyasıdır. Buna dosyaya da subl, atom vb. şekilde açtığınız klasörlerin kök dizininden ulaşabilirsiniz.

```bash
# ----------------------- #
#      Main Configs       #
# ----------------------- #

url: http://deneme.github.io
title: My Octopress Blog
subtitle: A blogging framework for hackers.
author: Your Name
simple_search: https://www.google.com/search
description:
```
Dosyaya ulaştığımızda öncelikle değiştirmemiz gereken url: kısmıdır çünkü repomuza hangi adı verdiysek o şekilde olmalıdır.

**Sitemizi Github'a Deploy Etme**

Yeni bir github repository oluşturmalıyız. Kişisel blogunuzu oluşturuyorsanız benim gibi aşağıdaki şekilde repo ismi verebilirsiniz.

```bash
esrefv.github.io
```
Sonrasında "rake setup_github_pages" çalıştırıyoruz,
```bash
$ bundle exec rake setup_github_pages
Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.io.git)
           or 'https://github.com/your_username/your_username.github.io')
Repository url: 
```
Örnek olarak belirttiği şekilde "git@github.com:username/username.github.com.git" kendi repo umuzu yazıyoruz.

Son olarak aşağıdaki kodları sırasıyla çalıştırarak blogumuzu Github'a deploy işlemimizi bitirmiş oluyoruz. 

```bash
$ bundle exec rake generate
$ git add .
$ git commit -m "First deploy to github." 
$ git push origin source
$ bundle exec rake deploy
```

Okuduğunuz için teşekkür ederim. 

Kaynak Siteler;

[http://octopress.org/](http://octopress.org/ "")
[https://daringfireball.net/projects/markdown/syntax](https://daringfireball.net/projects/markdown/syntax "")