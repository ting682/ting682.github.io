---
layout: post
title:      "Sinatra portfolio project"
date:       2020-07-21 13:12:50 +0000
permalink:  sinatra_portfolio_project
---


After some consideration of the topic of this Sinatra portfolio project, I decided on making a web app that records a user's prayer journal. I liked the idea that these days people are on their mobile phone or on their computer probably more than a physical book. I thought because of this, I should create a web app that has a prayer journal. Inside it, I wanted the user to have the option of entering their prayers, what is on their heart, answers to prayer, and to state what their thankful for. 

First, I heard about the [Corneal gem](https://github.com/thebrianemory/corneal). This was very helpful to generate all the base files and structure I needed in order to start at the meat of developing the app. I was very thankful for this gem to be available. 

During the development of this web app, Ayana Cotton--my very helpful instructor--helped point out to me a detail that I needed in my structure. For my model structure, I had a user, wallprayer, and journal class; however, in my journal class, Ayana pointed out to me that the naming convention might be a little bit off. I had a journal class which held my journal entries; however, I didn't want a user to have many journals. I intended to have a user have one journal with many entries. 

Because of this issue, I rewrote my code to be more specific. I did not want a class called "Journal" I wanted a class called "Journalentries". This is an important distinction because if someone after me were to view this web app, they would not understand that I want many journal entries and not many journals for my associations.  Even though it was a little painful to go back through my code and make the change, it was worth it in order to disambiguate my code.

Another issue I ran into was the idea of a timestamp. I started out thinking that I wanted to display a customized date. My table properties were the following:

```
    create_table :journalentries do |t|
      t.text :heart
      t.text :teachme
      t.text :prayer
      t.text :answer
      t.text :thankful
      t.string :date
      t.integer :user_id
    end
```

Here is how I created the journal entry below.

```
@date = Time.now.strftime("%b %d, %Y %I:%M %p")

@journalentry = Journalentry.create(heart: params[:heart], teachme: params[:teachme], prayer: params[:prayer], answer: params[:answer], thankful: params[:thankful], date: @date)
```

But later I watched Ayana's video showing that I could create the journal entry with the timestamps property. Now my create_table method looks like the following:

```
    create_table :journalentries do |t|
      t.text :heart
      t.text :teachme
      t.text :prayer
      t.text :answer
      t.text :thankful
      t.timestamps null: false
      t.integer :user_id
    end
```

And after this update, my schema is the following:

```
  create_table "journalentries", force: :cascade do |t|
    t.text     "heart"
    t.text     "teachme"
    t.text     "prayer"
    t.text     "answer"
    t.text     "thankful"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.integer  "user_id"
  end
```

Notice that now I have a created_at AND an updated_at column. This version is far superior from a customized date because now I have 2 separate columns and I don't have to upkeep. Activerecord automatically upkeeps this data as long as I add this column. Now creating my journal entry in my post route looks like this:

```
@journalentry = Journalentry.create(heart: params[:heart], teachme: params[:teachme], prayer: params[:prayer], answer: params[:answer], thankful: params[:thankful])
```

As mentioned before, I don't have to add a date field when creating my journal entry. Also, I can customize my date in both my show page and index page with this line of code:

```
<%= journalentry.created_at.localtime.strftime("%b %d, %Y %I:%M %p") %>
```

As a result, my date is formatted like the following: "Jul 20, 2020 12:36 PM". All of these changes helped me be more confident in my web app. If I desire, I can also create an updated_at field in my index and show page. I hope this was helpful. Happy coding!


