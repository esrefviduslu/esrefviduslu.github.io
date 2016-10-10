---
layout: post
title: "Rails counter_cache Kullanımı"
date: 2016-10-10 22:47:50 +0300
comments: true
categories: [rails, ruby, counter_cache]
---

Bugünkü yazımda Rails'ta counter_cache kullanımından bahsedeceğim. Öncelikle belli bir sayısal değeri kullanmamıza ihtiyaç varsa her seferinde count ile saydırmak yerine bunu counter _cache yardımı ile tutturabileceğiz. 

Kendi yaptığım proje üzerinden örnek vermem gerekirse. User ve Answer adında 2 modelim var. counter_cache kullanabilmemiz için 2 modelimizin arasında has_many, belogns_to ilişkisi olması gereklidir.

User modelim;

    class User < ActiveRecord::Base
        has_many :answers
    end

Answer modelim;

    class Answer < ActiveRecord::Base
        belongs_to :user
    end

şeklindedir.

Benim User'larımın birden çok Answer'ı olabilir. Answer içerindede User'ın online olup olmama durumunu, online offline olduğu saati ve buna benzer cevapları tutmaktayım.

Ben User'ıma ait cevapların sayını öğrenmem için 
    
    @user.answers.count
    @user.answers.size
    @user.answer.lenght 
şeklinde veritabanını yoracak COUNT(*) sorguları yerine daha efektif bir çözüm olan counter_cache kullanmam daha doğru olacaktır.

Öncelikle Answer modelimizdeki user'ı,

    class Answer < ActiveRecord::Base
        belongs_to :user, counter_cache: true
    end
şeklinde counter_cache: true yapıyoruz.

Burada belongs_to modelimeze counter_cache: true demiş olmamıza rağmen, User modelimize yeni bir alan eklemek için migration oluşturmamız gerekiyor.
 
```bash
$ rails g migration add_answers_sount_to_users    
```

Benim örneğimde görebileceğiniz gibi User modelimize answers_count eklememiz gerekiyor.

```ruby
class AddAnswersCountToUsers < ActiveRecord::Migration
  def change
    add_column :users, :answers_count, :integer, :default => 0
  end
end
```
Daha sonra yeni oluşturduğumuz migration ımızı migrate etmeliyiz.
```bash
$ bundle exec rake db:migrate   
```

Ve tamamdır. Her oluşacak yeni Answer'ımız hangi User'ımıza ait ise onun değerini 1 arttırıyor olacaktır. Bizde veritabanımızı yoracak COUNT(*) işlemi yerine. 

    @user.answers_count
    
yazarak, model tarafından birek ulaşabileceğiz.

Okuduğunuz için teşekkürler