---
layout: post
title:      "Prayer gatherings rails portfolio project"
date:       2020-09-03 16:16:25 +0000
permalink:  prayer_gatherings_rails_portfolio_project
---


I really had a lot of fun doing this rails portfolio project. My idea for my rails project was for a prayer gathering. A prayer gathering should have many attendees and should also have prayers to take to the prayer gathering. I also have a host that acts like an administrator and only a host can perform CRUD actions on the gathering. Any user can also create a prayer for the prayer gathering. Here are my relationships tables below 

![Prayer gathering associations](https://prayergathering.s3.us-east-2.amazonaws.com/Prayer+gathering+associations.JPG)

I had 2 areas that were challenging. One was to properly add an address to my gathering form. In this form, I had to create an address object with model. And looking back on it, this was a simple solution for a complex problem. So this is the code that I started with in regard to the address:

```
    def gathering_params
        params.require(:gathering).permit(:name, :meeting_date, :phone_number, :host_id, :url, :timezone, :address => [:address_1, :address_2, :city, :state, :zipcode])
    end
```

And this was my form below:

```
<%= form_for @gathering do |f| %>
<%= f.label :name %><br>
<%= f.text_field :name %><br><br>
<%= f.label :meeting_date %><br>
<%= f.datetime_select :meeting_date, default: Time.now.localtime, ampm: true %><br><br>
<%= f.label :timezone %><br>
<%= f.time_zone_select :timezone, ActiveSupport::TimeZone.us_zones %><br><br>
<%= f.label :phone_number %><br>
<%= f.text_field :phone_number %><br><br>
<%= f.hidden_field :host_id, :value => @host.id %>
<%= f.label :url, "URL" %><br>
<%= f.text_field :url %><br><br>
<%= f.fields_for :address, @address do |a| %>

    <%= a.label :address_1 %><br>
    <%= a.text_field :address_1 %><br><br>
    <%= a.label :address_2 %><br>
    <%= a.text_field :address_2 %><br><br>
    <%= a.label :city %><br>
    <%= a.text_field :city %><br><br>
    <%= a.label :state %><br>
    <%= a.select :state, options_for_select(us_states) %><br><br>
    <%= a.label :zipcode %><br>
    <%= a.text_field :zipcode %><br><br>
    
<% end %>

    <%= f.submit :class => "button" %><br><br>
<% end %>
```

I originally thought that as long as I can accept everything in the hash called address,  rails should generate an object called address; however, I quickly realized that rails did not permit the address field. After googling and playing around with the address object several rounds, I realized that I needed address_attributes instead of just address. Also, because my gathering object only had one address, the build method is different. It should be the following:


```
self.build_address(attributes)
```

After doing this and changing my strong params to the one shown below, I was finally able to move on.


```
    def gathering_params
        params.require(:gathering).permit(:name, :meeting_date, :phone_number, :host_id, :url, :timezone, :address_attributes => [:address_1, :address_2, :city, :state, :zipcode])
    end
```

So for my second major issue, it had to do with time validations. I had a datetime object created for the gathering meeting time. At first, my form looked like this:

```
<%= f.label :meeting_date %><br>
<%= f.datetime_select :meeting_date, default: Time.now.localtime, ampm: true %>
```

When I hit the create button, I realized that the datetime created was for UTC time and not the local time. After pulling my hair out, I realized that there was no way around this issue. I had 2 choices. One was to create a meeting time in UTC (which nobody understands other than software engineers), or I had to set a time zone. I realized that resolving this issue would be for my edification, so I decided to painfully add a column to my gatherings table called "timezone" and redo my form to look like this:

```
<%= f.label :meeting_date %><br>
<%= f.datetime_select :meeting_date, default: Time.now.localtime, ampm: true %><br><br>
<%= f.label :timezone %><br>
<%= f.time_zone_select :timezone, ActiveSupport::TimeZone.us_zones %><br><br>
```

Having a time_zone_select would help the user properly identify their timezone and allow me to adjust the timezone offset. My set_in_timezone method looked like this:

```
    def set_in_timezone(time, zone)
        Time.use_zone(zone) { time.to_datetime.change(offset: Time.zone.now.strftime("%z")) }
    end
```

Me gatherings controller looked like this:

```
    def create
        
        @host = current_user
        @gathering = @host.gatherings.build(gathering_params)
				@gathering.set_in_timezone(@gathering.meeting_date, @gathering.timezone)
				@gathering.save
    end    

```

There was no problem doing this until I ran into my update action in my controller. Here was the code.

```
    def update
        @host = current_user
        @gathering = Gathering.find(params[:id])
        #@gathering.meeting_date = set_in_timezone(@gathering.meeting_date, gathering_params[:timezone])
        
        if @gathering.update(gathering_params) && @gathering.address.save
            redirect_to gathering_path(@gathering)
        else
            flash[:error] = @gathering.errors.full_messages.to_sentence + @gathering.address.errors.full_messages.to_sentence
            render :edit
        end
        #binding.pry
        #@gathering.address.update(gathering_update_address_params)
    end
```

The problem here is that updating and validating happen in the @gathering.update method altogether! I found out that I can use the before_validation method. So in order to do this properly, I needed to reset the proper time before the validation. My code looked like the following:

```
    before_validation do
        self.meeting_date = set_in_timezone(self.meeting_date, self.timezone)
    end
```

After finally making that change, I was good to go. Also during validation, I checked to see that the meeting_date created was not in the past.This validation looked like the following:

```
    validate    :future_event
		
    def future_event
        if meeting_date != nil
           
            errors.add(:meeting_date, "cannot be in the past.") if self.meeting_date.in_time_zone < Time.zone.now
        end
        
    end    
```

After all these were implemented, I could finally move on. Happy coding!




