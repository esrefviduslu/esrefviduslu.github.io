---
layout: post
title: "Ruby ile Github-Bitbucket Arasında Çift Yönlü Repository Aktarımı Yapan Script"
date: 2016-11-05 19:00:59 +0300
comments: true
categories: [ruby, script, ruby script, github, bitbucket, repository]
---

Öncelikle başlık biraz uzun oldu arkadaşlar kusura bakmayın :D İlk olarak benden Bitbucket üzerinde bulunan projelerinin Github'a aktarma işlemi için bir script yazmam istendiğinde nasıl yapacağım konusunda en ufak bir fikrim yoktu ve ruby, rials dünyasında çok yeniyim. 

<!-- more -->

Sonrasında burada da yazağım gibi adım adım ilerleyerek mükemmel olmasada kullanılabilir bir script ortaya çıkartmaya çalıştım. Kodu bu yazıyı yazmadan önce bitirmiş olduğumdan elimden geldiğince iyileştirme yapmaya çalıştım fakat biraz daha temizlenip düzenlenebileceğine inanıyorum. Umarım işinizi görür.

#Repolarımızın bir dosyaya clone edilmesi

directoryname adını verdiğim bir değişkende oluşturulacak olan klasörün adını tutacağım ve rubyde 'Dir' sınıfını kullanacağız.
     
    directoryname = "bitbucketproject"
	Dir.mkdir(directoryname) unless File.exists?(directoryname)
	
Dir.mkdir sayesinde "bitbucketproject" adında klasörümüz oluşuyor. Ben ubuntu kulladığım için 'home' klasörü altında oluşuyor.

Klonlayacağımız repolarımızı bir repo_list adını verdiğimiz array içinde tutuyoruz. 
 
    repo_list = 
	["git clone git@github.com:github/example1.git",
	"git clone git@bitbucket.org:bitbucket/example2.git"]

Örnek olarak verdiğim listede terminal üzerinde nasıl clone işlemi yapıyorsak o şekilde yazıyoruz taşımak istediğimiz repoları.

Sonrasında 'directoryname' adını verdiğimiz klasörün içerisine tüm repolarımızı klonlayacak kodu yazıyoruz.

     repo_list.each do |repo|
		Dir.chdir(directoryname) do
			system repo	
		end
	end
	
repo_list içerisindeki tüm repolar bir bir klonlanıyor.

```ruby
    directoryname = "bitbucketproject"
	Dir.mkdir(directoryname) unless File.exists?(directoryname)
	
	
	repo_list = 
	["git clone git@github.com:github/example1.git",
	"git clone git@bitbucket.org:bitbucket/example2.git",
	"git clone git@bitbucket.org:bitbucket/example3.git",
	"git clone git@github.com:github/example4.git",
	"git clone git@bitbucket.org:bitbucket/example5.git",
	"git clone git@github.com:github/example6.git"
	]
	
    repo_list.each do |repo|
		Dir.chdir(directoryname) do
			system repo	
		end
	end
```

Genel olarak kodumuz bu şekildedir.

#Taşıyacağımız alandaki (Github || Bitbucket) remoteların listelenmesi

Nasıl klonlayacağımız repolarımızı listeleme işlemi yaptıysak aynı şekilde taşınacağı yerdede oluşturduğumuz repositoryleri aynı sırayla listemize ekliyoruz. (isimlerin aynı olma zorunluluğu yok)
    
    repo_list = 
	["git clone git@github.com:github/example1.git",
	"git clone git@bitbucket.org:bitbucket/example2.git"]
	
    remote_list = 
	["git remote add origin git@bitbucket.org:bitbucket/example1.git",
	"git remote add origin git@github.com:github/example2.git"]
	
Örnek olarak görebileceğiniz gibi klonlanacak repo_list arrayinde ilk sırada github/example1 var ve karşısında taşıyacağımız yeni repo bitbucket/example1.

Aynı şekilde sıralı olarak remote_list imizi oluşturuyoruz.

```ruby
	remote_list = 
	["git remote add origin git@bitbucket.org:bitbucket/example1.git",
	"git remote add origin git@github.com:github/example2.git",
	"git remote add origin git@github.com:github/example3.git",
	"git remote add origin git@bitbucket.org:bitbucket/example4.git",
	"git remote add origin git@github.com:github/example5.git",
	"git remote add origin git@bitbucket.org:bitbucket/example6.git"
	]
```
#Yeni remote ları eklemek için klonlanan dosyaları listeleme

Bu işlemin yapıla bilmesi için tek tek klasörlerin içerisine girilerek 'git' kodu çalıştırılması gerekiyor. 'directoryname' içerisinde bulunan tüm dosyaları listeleyen bir 'Dir' kodu var.

```ruby
	files = Dir.glob(directoryname + "/*")
	files_change = Dir.glob(directoryname + "/*")
```
'Dir' glob belirttiğimiz path içerisindeki dosyaları listeliyor bize. Fakat listeleme tarzı 'bitbucket/example1' şeklinde olduğu için tek tek içerisini gezme açısında güzel, ayrıca bize sadece dosya adı olan 'example1' de lazım olduğundan neden iki tane kullanıyoruz diye düşünebilirsiniz, sıra onda :)

```ruby
	files_change.each do |file_change|
		file_change.slice! directoryname + "/"
	end
```
Bu kod sayesinde files_change içerisinde sadece klasör isimlerini tutuyoruz. Yaptığı işlem ise 'bitbucket/example1' şeklinde olan listedeki verilerin önünden 'bitbucket/' ı siliyor ve bize sadece klasör isimlerini bırakıyor.

#Yeni bir remote listesi oluşturma zorunluluğu

Genel olarak tekli ve ikili olarak test ettiğim bu scriptte çoğul bir deneme yaptığımda klonlanan reponun klasör isminin baş harfine göre bir sapma yaşanıyor ve klonlanan repo başka bir repoya pushlanabiliyordu. 

Fakat merak etmeyin kullanıcı sadece repo_list ve remote_list i sırayla girdikten sonra başka bir işlem yapmasına gerek kalmadan biz sıralamayı kodla halledeceğiz.

```ruby
    new_remote_list = Array.new
	order_count = 0
	repo_count = 0	
	files_change.each do |fileschange|	
		repo_list.each  do |repolist| 
			if repolist.include? fileschange
				new_remote_list[repo_count] = remote_list[order_count]
				repo_count += 1
			end
			order_count += 1
		end
		order_count=0	
	end
```

Kodumuz bu şekilde. Nasıl çalıştığına gelirsek. files_change bizim 'example1' şeklinde tuttuğumuz klonlanan repolarımızın dosya adı listesi. Ve files_change klasörlerimizi okuduğu sıralamayla bize dosya adı listesi yaratıyor. Bu önemli!! repo_list en başta anlattığım  klonlanacak olan repolarımızın sıralı listesi. 

```ruby
    files_change.each do |fileschange|
    
    end
```
Bu blokta klasörlerin okunduğu sırada dosyaları gezeceğiz. 

```ruby
    files_change.each do |fileschange|	
		repo_list.each  do |repolist| 
			
		end
		order_count=0	
	end
```
repo_list te ise ilk sırada klonlamak istediğimiz sırada girdiğimiz repository listemiz ve bunları gezeceğiz. Neden mi? Çünkü bu sıra bizim taşıyacağımız yer için kullacağımız remote_list ile aynı olan sıra ve ilk başta dediğim gibi taşıyacağınız yerin repository adı aynı olmak zorunda değil.


	
repo_list in ilk sırasında "github/example1.git" var bu klonladığımız repository. Klasörümüzün adı ise example1 , remote_list imizde ise ilk sırada "bitbucket/example1.git" var buda yeni göndereceğimiz repository. 
```ruby
    repo_list.each  do |repolist| 
			if repolist.include? fileschange
				new_remote_list[repo_count] = remote_list[order_count]
				repo_count += 1
			end
			order_count += 1
		end
```
Belirttiğim gibi klon aşamasında klasör isimlerine göre sıralama değişiyor. if koşulu -> repo_list in ilk sırasındaki veride "files_change" klasör adı geçiyormu diye bakıyoruz. Eğer geçiyorsa new_remote_list[0] dan başlayarak eşleşmeleri içerisine ekle. 

Ve sonunda new_remote_list olarak adlandırdığımız okunan klasör sırasına göre yeni bir remote listemiz oluştu.

Örnek ;

    repo_list = 
	["git clone git@github.com:github/example1.git", ...]
	fileschange = "example1"
	
    "git clone git@github.com:github/example1.git" içerisinde "example1" geçiyormu?
    
Şeklinde files_change okunan dosyaların listesi olduğu için ilk sırada 'example4' te olabilirdi.

#Yeni remote ları repolarımıza ekleyelim

Bir projelerimizi commitleri ve eğer master - develop brachleri varsa o şekilde pushlayacağız. Klonlanan projelerin 'origin remote' ları klonlanan yer neresi ise orası kalıyor. Biz onuda silmeden origin_two isminde yeni bir remote ekleyeceğiz.

```ruby
    new_remote_list.each do |new_remote|	
		new_remote.gsub! 'origin', 'origin_two'
	end
```
Remote listemizi 'origin_two' olacak şekilde düzenliyoruz.
```ruby
	remote_count = 0
	files.each do |file|
		Dir.chdir(file) do			
			remote = new_remote_list[remote_count]						
			system 'git remote -v'
			system remote
			system 'git remote -v'					
		end
		remote_count +=1
	end
```
Bu kodda tek tek dosyalarımıza girerek 'git remote add origin_wo ' kodunu çalıştıracak. 'git remote -v' ise eklenmeden önceki ve sonraki halini ekrana basacak.

#Yeni repomuza pushluyoruz ve origin_two remoteunu siliyoruz

```ruby
	files.each do |file|
		Dir.chdir(file) do
			system 'git checkout master'
			system 'git branch -a'				 
			system 'git push origin_two master'
			system 'git checkout develop'
			system 'git branch -a'
			system 'git push origin_two develop'				
		end		
	end
	
	remote_remove = 0
	files.each do |file|
		Dir.chdir(file) do
			remote = remote_list[remote_remove]
			system 'git remote -v'
			system 'git remote remove origin_two'	
		end
	remote_remove += 1			
	end
```

Bu kodda ilk döngüde öncelikle master branche geçiyoruz. Ve yeni remote umuza 'origin_two' ya masterı pushluyoruz. Sonrasında develop branch için aynı işlemi uyguluyoruz.

İkinci döngüde ise 'git remote remove' ile origin_two remoteumuzu kaldırıyoruz. Aslında kaldırmamızı gerektiren bir durum yok isterseniz kaldırmaya bilirsiniz. 

Elimden geldiğince açıklamaları ile anlatmaya çalıştım. Yazı biraz fazla uzun oldu başlık gibi :D Okuduğunuz için teşekkürler.