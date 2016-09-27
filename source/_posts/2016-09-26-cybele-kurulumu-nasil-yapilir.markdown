---
layout: post
title: "Cybele Kurulumu Nasıl Yapılır?"
date: 2016-09-26 11:06:43 +0300
comments: true
categories: [rails, ruby]
---
Cybele "Ruby on Rails" dünyasına yeni başlayanlar için(biride benim) çok kolay proje oluşturma ve user-admin arayüzleri sayesinde her şeyi daha rahat yönetebilmemize olanak sağlayan bir şablondur.

Kurulum aşamasından önce gerekli olan en düşük versiyon tipleri;

> Ruby  ~> 2.3

> Rails ~> 4.2 'dir.

Eğer daha düşük bir versiyon kullanmaktaysanız öncelikle güncelleme yapmalısınız.
<!-- more -->

Öncelikle "Cybele" gem imizi indirmekle başlayalım.

```bash
$ gem install cybele
```
Daha sonra ise indirdiğimiz "Cybele" ile proje oluşturmak için terminalimize

```bash
$ cybele project_name
```
project_name yerine istediğimiz bir proje adını kullanabiliriz. Ben bu anlatımım sırasında adını "cybele_test" olarak kullanacağım.


```bash
$ cybele cybele_test
```
diyerek projemizi oluşturmuş oluyoruz. Fakat öncelikle halletmemiz gereken bir kaç düzenleme daha var.

Oluşturduğumuz projemizin dizinine girerek rubymine, atom yada subl vb. ne kullanıyorsanız onunla projemizi açıyoruz.

cybele_test projemizin altında **db/migrate** klasörünün içerisinde bulunan **devise_create_user** ve **devise_create_admin** dosyalarında değiştirmemiz gereken alanlar bulunmakta.

```ruby
t.boolean :is_active
t.string :time_zone
```
Her iki dosyanın içerisinde de yukarıdaki gibi bulunan **is_active** ve **time_zone** satırlarının,

```ruby
t.boolean :is_active, default: true
t.string :time_zone, default: "UTC"
```
bu şekilde default değerlerini ayarlıyoruz.

Projemizin içerisindeki public klasörüne,

```bash
$ cd public
```
yazarak ulaşıyoruz ve içerisinde aşağıdaki komutu çalıştırıyoruz.

```bash
$ ln -s ../VERSION.txt VERSION.txt
```
Bu işlemide gerçekleştirdiysek sırada veritabanı işlemlerimizi gerçekleştirmek kaldı.

Projemizin config klasörünün altındaki database.yml uzantılı dosyamızı açıyoruz. **(config/database.yml)**

```ruby
development: &default
adapter: postgresql
database: cybele_test_development
encoding: utf8
min_messages: warning
pool: 5
timeout: 5000
host: localhost
port: 5432
```
Yukarıdaki şekilde görünen development kısmımızın altına username ve password ekleyeceğiz.

```ruby
development: &default
adapter: postgresql
database: cybele_test_development
encoding: utf8
min_messages: warning
pool: 5
timeout: 5000
host: localhost
port: 5432
username: cybele_test
password: cybele_test
```
Postgresql'de bu klasörde gördüğümüz şekilde database isminin aynısını oluşturarak üzerine username ve password tanımlamam gerekiyor. Bunun için öncelikle;

```bash
$ sudo -u postgres psql
```
kullanarak postgresql terminalini açıyoruz.

```bash
$ CREATE DATABASE ‘cybele_test_development’;
```
Yazarak veri tabanımızı oluşturuyoruz. database.yml dosyamızda database ismimiz nasılsa öyle yazmamız gerekiyor.

```bash
$ CREATE USER cybele_test PASSWORD 'cybele_test';
```
database.yml dosyamızda belirttiğimiz şekilde username ve password ümüzü tanımlıyoruz.

```bash
$ ALTER USER cybele_test WITH SUPERUSER;
```
yazarak veri tabanı işlemlerimizi bitirmiş oluyoruz.

Son olarak yapmamız gereken ise aşağıdaki kodları sırasıyla terminalde çalıştırmak,

```bash
$ bundle exec rake db:create
$ bundle exec rake db:migrate
$ budnle exec rake dev:seed
```

ve

```bash
$ rails s
```

yazarak projemizi çalıştırmış oluyoruz. [http://localhost:3000/](http://localhost:3000/ "") e giderek projemizin çalışan halini görebiliriz.

Başlangıçta belirttiğim gibi bir tanede Admin sayfamız bulunmakta. Admin olarak giriş yapmak için ise [http://localhost:3000/hq](http://localhost:3000/hq "") sayfasına gitmeniz gerekiyor.

Adminin e-posta ve parolası **db/seeds.rb** dosyasında yazıyor olacaktır.

Okuduğunuz için teşekkürler.
