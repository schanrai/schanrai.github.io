---
layout: post
title:      "My Rails Project: Building the right data foundation with Faker"
date:       2019-09-27 13:07:55 -0400
permalink:  my_rails_project_building_the_right_data_foundation_with_faker
---


Vanilla Forum is a standalone  app for Rails that aims to be a basic no-frills forum web application. My goal was to create an application that provides the basic functionality of forums, topics and posts with some of the more popular content and social features of platforms such as **Quora** and **Reddit**.

For this project, I took at least twice the time in planning as I had on previous projects  to ensure I kept on track. It is all too easy to blow the scope way out of water when you’re working solo and at your own pace, so it was important for me to document all the main requirements grouped by their parent model.  It was also necessary that I create the right kind of data environment since the spec included querying and scope functions. 

Beyond describing basic CRUD functions, I noted any conditions or criteria impacting them and mapped out my relationships between my model via an **ERD**:


![](https://i.ibb.co/XbFngMd/ERD.png)

```
User: (username, email, password,uid) *use uid for omniauth provider :uid, limit: 30
has_many :posts
has_many :threads, :through =>:posts
has_many :upvotes
 
Thread: (subject)
has_many :posts
has_many :users, :through =>:posts
 
Post: (content, user_id, thread_id)
belongs_to :user
belongs_to :threads
has_many :upvotes
 
Upvotes: (user_id, post_id)
belongs_to : user
belongs_to :post
```

![](https://i.ibb.co/MgbK1Zj/Screen-Shot-2019-09-27-at-11-48-27-AM.png)

Armed with a bunch of great resources to get me started, (check out Rails Section Lead -**Jennifer Hansen**'s awesome instructional videos [here](https://www.youtube.com/channel/UCfuRggLR31GXr1ZBOSdvKiQ)) I felt confident and prepared to knock this project out in no time. Boy, was I in for a ride!

Nothing could have prepared me for the sheer volume and range of things I learnt along the way whilst making mistakes. You will always learn more from making your own mistakes, and one of the biggest takeaways from this project was to pay close attention to how your data is initially created. Having your business rules and logic mapped out clearly from the outset and baked into your test/seed data can prevent a lot of hair-pulling and long nights later down the line. 

## On Faker time

[Faker](https://github.com/faker-ruby/faker) is a library that generates random/fake data in your database. The prospect of getting real-looking test data instantly, and having my database populated with more than one or two records  when building and testing the app resonated highly with me– and so I gleefully installed the gem and accepted a dinner invitation for that evening with the intention that my database would be seeded by then.

In reality, it took me a whole day to get the data looking right! Let me walk you through my journey...

Initially, I referenced t articles by **[Matthew Masiello](https://medium.com/@sotek222/seed-your-database-with-fakes-using-faker-5ba5dccda44f)** and **[Yassi Moretenson](https://medium.com/@yassimortensen/using-faker-to-seed-your-rails-database-cbfb0960d573)** to help me with installation and initial configuration. The READMe [documentation](https://github.com/faker-ruby/faker) is concise and covers most areas at high level,  but, the real learning came through trial and error.

I would suggest that if you are using this gem for the first time, to seed in increments by your models. In this way, you can evaluate if the criteria/specifiers you have set up for each generator work for your application and adjust accordingly. Just delete the info in the seed file once you have run the `rake db:seed` command (as many times as desired) and you have populated the database according to your requirements.

It is important to understand, that every time you run the `rake db:seed` command, you will execute the methods/commands in your seed file. For instance, if you run the following block of code with the `rake db:seed` call once, it will generate 3 distinct user records in your database with name, email and password attributes set.

```
3.times do
    User.create(
        name: Faker::Name.username,
        email: Faker::Internet.email,
        password: Faker::Internet.password
)
end

```

If nothing changes in the seed file and you run it twice more (a total of three times) you would have 9 users seeded in the database. Staggering the seeding is also useful if you’re planning to build any time-based queries or methods in your application, by not creating your records all at the same time you don’t have to drill down to milliseconds to see the difference between the records when writing and testing chronological sorting, filtering or ordering functions.

## Respect your own business rules

One thing to keep in mind is  **Faker generates data at random every time you call a method on it’s library of data**. So if you have to seed you database more than once, you are not guaranteed the same data the second time. Conversely, the returned values of each method call are not guaranteed to be unique by default. Even with validations on your models you cannot guard against erroneous or duplicate data when you seed with Faker.

Ther may also be instances or times when your business and data rules might get compromised - no level of specification and fine-tuning the generator can account for that. For instance, in the environment of my ForumApp, a user cannot create more than one upvote on a post. This means that when I seeded the data, it created duplicate upvotes from the same user on some posts.

I would recommend having a good visual, DB tool to view and edit database records when you are creating data with Faker. I used [DB Browser for  SQLite](https://sqlitebrowser.org//) which allows you to sort columns in tables easily and view all your data in tables at a glance, a feature that proved to be invaluable for me in identifying duplicates in the generated data.

![](https://i.ibb.co/Y2tXvmq/Screen-Shot-2019-09-27-at-12-40-46-PM.png)

Another example of not heeding my own business rules when creating my dataset was when,during generation of the seed data, I generated more topic than post data. I planned to implement the functionality of new post creation in such a way as to make it mandatory at time of initialization to associate it with a topic. However, due my oversight there were a bunch of records in the database with topic info only and no associated posts.

Whilst this oversight seemed innocuous in the early stages of the build, it caused me a whole day trying to chase down/isolate a bug on my Threads Homepage in the home-stretch!.

The cracks revealed themselves upon this block of code:

```
<% @topics.each do |topic| %>
    	<h5 ><%= link_to("#{topic.title}", topic_path(topic.id)) %></h5>
        	<% posty = topic.posts.oldest%>
       	  <%= "posty.user.username" %>

```

Iterating and exposing the values of the associated user model’s attributes seemed to be no issue in the console whenever I stopped the code to interrogate with a pry. However, in the view, I kept getting the error `NoMethodError: undefined method ‘user' for nil:NilClass` for the following line of code:

```
<%= posty.user.username %>
```

Why was nothing returning for `posty` in the view? Why was I not able to use any methods provided by the Activerecord associations to retrieve data values about the items I was iterating over?


After a consultation wth Jennifer, I realized that errors were being thrown because there was indeed nothing encapsulated in `posty` at various points in the loop where the thread had no associated posts. She gave me a handy tip to leverage the more forgiving nature of [try](https://apidock.com/rails/v3.2.8/Object/try). This would prevent raising an ugly exception message anytime `posty` had no data, it would just return nil. Thus, I refactored the rest of my methods in the data loop as follows, so that some threads could return with no post data front end and not throw errors:

```
<%= posty.try(:user).try(:username) %> 
```


![](https://i.ibb.co/0Xg6gGH/Thread-no-posts.png)

This yielded a much more elegant solution than I could ever have imagined. The likelihood of such an edge case happening again wherein a post could be created without a topic were already accounted for in the form designs was negligible, as new post creation is not possible without creation or association to a topic.


## The skinny

Overall, Faker proved to not be the time-saving utility I hope it would be on this particular occasion. However, I attribute this to my own inexperience with the library and a lack of 'real life' documentation readily available through Google searches (via articles, blog posts etc). It is my hope to begin changing that with this blog post.

I believe that you can leverage it to save development time if you use it judiciously, paying close attention to your business logic and it's impacts on your data rules.


You check out my project via this [video walkthrough](https://youtu.be/1I2UA-P8Md0) or find my repo [here](https://github.com/schanrai/VanillaForum).




