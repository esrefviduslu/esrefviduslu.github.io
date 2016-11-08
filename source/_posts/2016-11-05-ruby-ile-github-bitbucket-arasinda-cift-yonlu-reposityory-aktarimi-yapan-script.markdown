---
layout: post
title: "Ruby ile Github-Bitbucket Arasında Çift Yönlü Repository Aktarımı Yapan Script"
date: 2016-11-05 19:00:59 +0300
comments: true
categories: [ruby, script, ruby script, github, bitbucket, repository]
---
Öncelikle başlık biraz uzun oldu arkadaşlar kusura bakmayın :D İlk olarak benden Bitbucket üzerinde bulunan projelerinin Github'a aktarma işlemi için bir script yazmam istendiğinde nasıl yapacağım konusunda en ufak bir fikrim yoktu ve ruby dünyasında çok yeniyim. 

<!-- more -->

Sonrasında burada da yazağım gibi adım adım ilerleyerek mükemmel olmasada kullanılabilir bir script ortaya çıkartmaya çalıştım. Kodu bu yazıyı yazmadan önce bitirmiş olduğumdan elimden geldiğince iyileştirme yapmaya çalıştım fakat biraz daha temizlenip düzenlenebileceğine inanıyorum. Umarım işinizi görür.

#Var olan repositorylerimizin, taşınacak alandaki remoteların listelenmesi 

directoryname adını verdiğim bir değişkende oluşturulacak olan klasörün adını tutacağım ve rubyde 'Dir' sınıfını kullanacağız.
     
    directoryname = "bitbucketproject"
	Dir.mkdir(directoryname) unless File.exists?(directoryname)
	
Dir.mkdir sayesinde "bitbucketproject" adında klasörümüz oluşuyor. Ben ubuntu kulladığım için 'home' klasörü altında oluşuyor.

Klonlayacağımız repolarımızı ve taşıyacağımız alandaki repositorylere ait remotelar repo_remote_list adını verdiğimiz array içinde tutuyoruz. 
 
    repo_list = 
	[["git clone git@github.com:github/example1.git", "git clone git@github.com:github/example1.git"],
	["git clone git@bitbucket.org:bitbucket/example2.git","git clone git@bitbucket.org:bitbucket/example2.git"]]

Örnek olarak verdiğim listede terminal üzerinde nasıl clone işlemi yapıyorsak o şekilde yazıyoruz taşımak istediğimiz repoları, taşıyacağımız alana ait remoteları da ikili liste şeklinde hangi repomuzu hangi 'remote' a taşımak istiyorsak yanına belirtiyoruz.


```ruby
    	directoryname = "bitbucketproject"
	Dir.mkdir(directoryname) unless File.exists?(directoryname)
	 
	repo_remote_list = 
	[
	["git clone git@github.com:github/example1.git", "git remote add origin git@bitbucket.org:bitbucket/example1.git"],
	["git clone git@bitbucket.org:bitbucket/example2.git", "git remote add origin git@github.com:github/example2.git"],
	["git clone git@bitbucket.org:bitbucket/example3.git", "git remote add origin git@github.com:github/example3.git"],
	["git clone git@github.com:github/example4.git", "git remote add origin git@bitbucket.org:bitbucket/example4.git"],	
	["git clone git@bitbucket.org:bitbucket/example5.git", "git remote add origin git@github.com:github/example5.git"],
	["git clone git@github.com:github/example6.git", "git remote add origin git@bitbucket.org:bitbucket/example6.git"]
	]
	
```

Genel olarak kodumuz bu şekildedir.

#Var olan repositorylerimize 'mirror' eklenmesi ve klasör içine klonlama işlemi

Bu aşamada 'mirror' işlemi bize, yeni alana taşıyacağımız repositorylerimizin tüm branchleri ile taşımıza yardımcı olacak olan büyülü bir kelime.(Bende yeni öğrendim ve kullanıyorum :D)

Bunun kullanılma şekli klon aşamasında kodumuz şu şekilde olmalı,

    "git clone --mirror git@github.com:github/example1.git"
fakat ben kullanıcının bunu yazarak zaman kaybetmesini istemediğim için kullanıcı sadece clone repoyu ve remote repoyu yazacak gerisini biz halledeceğiz. 

```ruby
	repo_count = 0 
	repo_remote_list.each do |repo|
		repo_first = repo[0].rpartition(' ').first << " --mirror "  
		repo_last  = repo[0].rpartition(' ').last
		repo_remote_list[repo_count][0] = repo_first + repo_last
		repo_count += 1
	end
```

Onun içinde bu şekilde bir kod yazıyoruz. Bu 'rpartition(' ') kodu,
    
    'git clone'  
    'git@github.com:github/example1.git' 

şeklinde klonalanacak olan repomuzu ikiye bölüyor. Bizim ise ilk bölümün yanına " --mirror " büyülü sözcüğünü eklemek. Bu işlemide yaparak repo_remote_list imizi yenilemiş oluyoruz. Şimdide klonlayalım.

```ruby
	repo_remote_list.each do |repo|
		Dir.chdir(directoryname) do
			system repo[0]	
		end
	end
```

#Yeni remote ları eklemek için klonlanan dosyaları listeleme

Bu işlemin yapıla bilmesi için tek tek klasörlerin içerisine girilerek 'git' kodu çalıştırılması gerekiyor. 'directoryname' içerisinde bulunan tüm dosyaları listeleyen bir 'Dir' kodu var.

```ruby
	files = Dir.glob(directoryname + "/*")
	files_change = Dir.glob(directoryname + "/*")
```
'Dir' glob belirttiğimiz path içerisindeki dosyaları listeliyor bize. Fakat listeleme tarzı 'direcktoryname/filename' bizde => 'bitbucketproject/example1' şeklinde olduğu için tek tek içerisini gezme açısında güzel, ayrıca bize sadece dosya adı olan 'example1' de lazım olduğundan neden iki tane kullanıyoruz diye düşünebilirsiniz, sıra onda :)

```ruby
	files_change.each do |file_change|
		file_change.slice! directoryname + "/"
	end
```
Bu kod sayesinde files_change içerisinde sadece klasör isimlerini tutuyoruz. Yaptığı işlem ise 'bitbucketproject/example1' şeklinde olan listedeki verilerin önünden 'bitbucketproject/' ı siliyor ve bize sadece klasör isimlerini bırakıyor.

#Yeni bir remote listesi oluşturma zorunluluğu

Genel olarak tekli ve ikili olarak test ettiğim bu scriptte çoğul bir deneme yaptığımda klonlanan reponun klasör isminin baş harfine göre bir sapma yaşanıyor ve klonlanan repo başka bir repoya pushlanabiliyordu. 

Fakat merak etmeyin kullanıcı sadece repo_remote_list i sırayla girdikten sonra başka bir işlem yapmasına gerek kalmadan biz sıralamayı kodla halledeceğiz.

```ruby
    new_repo_remote_list = Array.new
	new_repo_count = 0	
	files_change.each do |fileschange|
		repo_remote_list.each  do |repolist|
			if repolist[0].include? fileschange
				new_repo_remote_list[new_repo_count] =  repolist[1] 
				new_repo_count += 1
			end
		end
	end
```

Kodumuz bu şekilde. Nasıl çalıştığına gelirsek. files_change bizim 'example1' şeklinde tuttuğumuz klonlanan repolarımızın dosya adı listesi. Ve files_change klasörlerimizi okuduğu sıralamayla bize dosya adı listesi yaratıyor. Bu önemli!! repo_remote_list 'repolist[0]' klonlanacak olan repolarımızın sıralı listesi. 

```ruby
    files_change.each do |fileschange|
    
    end
```
Bu blokta klasörlerin okunduğu sırada dosyaları gezeceğiz. 

```ruby
    files_change.each do |fileschange|
		repo_remote_list.each  do |repolist|
			if repolist[0].include? fileschange
			    
			end
		end
	end
```
repolist[0]  ise ilk sırada klonlamak istediğimiz sırada girdiğimiz repository listemiz ve bunları gezeceğiz. Neden mi? Çünkü bu sıra bizim taşıyacağımız yer için kullacağımız 'remote' lar ile aynı olan sıra ve taşıyacağınız yerin repository adı aynı olmak zorunda değil.
	
repolist[0] in ilk sırasında "github/example1.git" var bu klonladığımız repository. Klasörümüzün adı ise example1 , repolist[1] imizde ise ilk sırada "bitbucket/example1.git" var buda yeni göndereceğimiz repository. 
```ruby
    files_change.each do |fileschange|
		repo_remote_list.each  do |repolist|
			if repolist[0].include? fileschange
				new_repo_remote_list[new_repo_count] =  repolist[1] 
				new_repo_count += 1
			end
		end
	end
```
Belirttiğim gibi klon aşamasında klasör isimlerine göre sıralama değişiyor. if koşulu -> repolist[0] in ilk sırasındaki veride "files_change" klasör adı geçiyormu diye bakıyoruz. Eğer geçiyorsa new_repo_remote_list[new_repo_count] dan başlayarak eşleşmeleri içerisine ekle. 

Ve sonunda new_remote_list olarak adlandırdığımız okunan klasör sırasına göre yeni bir remote listemiz oluştu.

Örnek ;

    repo_list = 
	["git clone git@github.com:github/example1.git", ...]
	fileschange = "example1"
	
    "git clone git@github.com:github/example1.git" içerisinde "example1" geçiyormu?
    
Şeklinde files_change okunan dosyaların listesi olduğu için ilk sırada 'example4' te olabilirdi.

#Yeni remote ları repolarımıza ekleyelim

Biz projelerimizi tüm commitleri ve varsa tüm brachleri ile pushlamayı istiyoruz. Klonlanan projelerin 'origin remote' ları klonlanan yer neresi ise orası kalıyor. Biz onuda silmeden origin_two isminde yeni bir remote ekleyeceğiz.

```ruby
    new_repo_remote_list.each do |new_remote|	
		new_remote.gsub! 'origin', 'origin_two'	
	end
```
Remote listemizi 'origin_two' olacak şekilde düzenliyoruz.
```ruby
	remote_count = 0
	files.each do |file|
		Dir.chdir(file) do			
			remote = new_repo_remote_list[remote_count]						
			system 'git remote -v'
			system remote
			system 'git remote -v'					
		end
		remote_count +=1
	end
```
Bu kodda tek tek dosyalarımıza girerek 'git remote add origin_two ' kodunu çalıştıracak. 'git remote -v' ise eklenmeden önceki ve sonraki halini ekrana basacak.

#Yeni repomuza pushluyoruz ve origin_two remote unu siliyoruz

```ruby
	files.each do |file|
		Dir.chdir(file) do			 
			system 'git push --all origin_two'				
		end		
	end
	
	remote_remove = 0
	files.each do |file|
		Dir.chdir(file) do
			remote = new_repo_remote_list[remote_remove]
			system 'git remote -v'
			system 'git remote remove origin_two'	
		end
	remote_remove += 1			
	end
```

Bu kodda ilk döngüde o anda var olan tüm branchleri pushluyoruz.

İkinci döngüde ise 'git remote remove' ile origin_two remoteumuzu kaldırıyoruz. Aslında kaldırmamızı gerektiren bir durum yok isterseniz kaldırmaya bilirsiniz. 

Elimden geldiğince açıklamaları ile anlatmaya çalıştım. Yazı biraz fazla uzun oldu başlık gibi :D Okuduğunuz için teşekkürler. 

Tam koda [buradan](https://gist.github.com/esrefv "") ulaşabilirsiniz adı ise "reposync.rb" direk kodun sayfasını veremiyorum çünkü istenen bir değişiklik yada güncelleme olduğunda burada sayfayı güncellemem gerekiyor bu şekilde daha kolay ulaşabilirsiniz. İyi günler. 