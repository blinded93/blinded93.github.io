---
layout: post
title: "Rails Calendar"
date:       2019-10-03 22:31:45 -0400
permalink: rails-calendar
---

During my time as a student of Flatiron, we were tasked with creating 5 projects in each of the separate modules. My Rails project was called HouseholdManagement. As was said in my project completion post, the idea came to be from a need to organize, better understand and visualize aspects of running a household. One of these visualization tools I used was a Calendar. My goal was to make it dynamic, able to display the data in a way that was easy to understand and quick to navigate.

I chose to implement an elegant and easy to modify calendar from Railscast #213 that was quickly adapted to my use case. Following is a look at the code from the walkthrough, which I will break down afterward.

*/app/controllers/calendar.rb*
```ruby
class CalendarController < ApplicationController
  def show
    @date = params[:date] ? Date.parse(params[:date]) : Date.today
  end
end
```

*/app/views/show.html.erb*
```ruby
<div class='row'>
  <div class='col-md-12 text-center'>
    <div class='well controls'>
      <%= link_to calendar_path(date: @date.prev_month), class: 'btn btn-default' do %>
        <i class='glyphicon glyphicon-backward'><</i>
      <% end %>
      <%= "#{@date.strftime("%B")} #{@date.year}" %>
      <%= link_to calendar_path(date: @date.next_month), class: 'btn btn-default' do %>
        <i class='gliphicon glyphicon-forward'>></i>
      <% end %>
    </div>
  </div>

  <div class='row'>
    <div class='col-md-12'>
      <%= calendar @date do |date| %>
        <%= date.day %>
      <% end %>
    </div>
  </div>
</div>
```

*/app/helpers/calendar.rb*
```ruby
module CalendarHelper
  def calendar(date = Date.today, &block)
    Calendar.new(self, date, block).table
  end
end
```

*/lib/calendar.rb*
```ruby
class Calendar
  HEADER = %w[Sun Mon Tue Wed Thur Fri Sat]
  START_DAY = :sunday

  delegate :content_tag, to: :view

  attr_accessor :view, :date, :callback

  def initialize(view, date, callback)
    @view = view
    @date = date
    @callback = callback
  end

  def table
    classes = "calendar table table-bordered"
    content_tag :table, class: classes do
      header + week_rows
    end
  end

  def header
    content_tag :tr, class:'bg-info' do
      HEADER.map { |day| content_tag :th, day, class:'text-center p-1' }.join.html_safe
    end
  end

  def week_rows
    weeks.map do |week|
      content_tag :tr, class:'bg-light' do
        week.map { |day| day_cell(day) }.join.html_safe
      end
    end.join.html_safe
  end

  def weeks
    first = date.beginning_of_month.beginning_of_week(START_DAY)
    last = date.end_of_month.end_of_week(START_DAY)
    (first..last).to_a.in_groups_of(7)
  end

  def day_cell(day)
    content_tag :td, view.capture(day, &callback), class: day_classes(day)
  end

  def day_classes(day)
    classes = ['day-cell']
    classes << 'table-success' if day == Date.today
    classes << 'table-active' if day.month != date.month
    classes.join(' ')
  end
end
```

##### The Breakdown
When the ‘/calendar’ path is visited, the **calendar**#**show** action is executed which assigns a default date of today to @date and renders the calendar show view.

*/app/controllers/calendar.rb*
```ruby
class CalendarController < ApplicationController
  def show
    @date = params[:date] ? Date.parse(params[:date]) : Date.today
  end
end
```

The calendar view is broken down into two rows:

*/app/views/show.html.erb*
```ruby
<div class='col-md-12 text-center'>
  <div class='well controls'>
    <%= link_to calendar_path(date: @date.prev_month), class: 'btn btn-default' do %>
      <i class='glyphicon glyphicon-backward'><</i>
    <% end %>
    <%= "#{@date.strftime("%B")} #{@date.year}" %>
    <%= link_to calendar_path(date: @date.next_month), class: 'btn btn-default' do %>
      <i class='gliphicon glyphicon-forward'>></i>
    <% end %>
  </div>
</div>
```
The first contains the Month name and links on either side to change the month forward or back. These links are routed to the calendar path with a date URL parameter, changing the @date in the controller and reloading the calendar#show template with the updated date and different month. In my app, I used the remote: true option in the links to prevent a complete reload. My show page was in a partial, so from the controller, I changed the HTML using Javascript to the newly rendered partial.

*/app/views/show.html.erb*
```ruby
<div class='row'>
    <div class='col-md-12'>
      <%= calendar @date do |date| %>
        <%= date.day %>
      <% end %>
    </div>
  </div>
```
The second row renders a table using the helper method called calendar. This method is being called with two parameters, the @date variable given by the CalendarController and the block that is passed to the method. In this basic example, the 'date.day' is passed as the block, which is the day of the month.

Now that the view is understood, let’s move on to the #calendar helper method that was called earlier. This method is relatively straightforward, instantiating a new Calendar object and calling the table instance method on it. The passed parameters are ‘self’, ‘date’ and the block. When looking closer at self in the Helper module, you’ll see this object is connected to many other objects and has access to 447 methods! Though I was having trouble finding exactly how those objects and methods were connected through all of the Rails magic, the important thing to note is the view methods we will be calling on this object: #content_tag and #capture.

On to the Calendar class! The walkthrough suggested inheriting from Struct.new:
```ruby
class Calendar <Struct.new(:view, :date, :callback)
end
```

I just made it a regular class.

After creating the class there are two class constants: HEADERS and START_DAY.
```ruby
HEADER = %w[Sun Mon Tue Wed Thur Fri Sat]
START_DAY = :sunday
```

Changing the former will modify the first row of the calendar table (the day names), while the latter is responsible for the starting day of each week row.

Next is the delegate call:
```ruby
delegate :content_tag, to: :view
```

Remember the view methods from the helpers ‘self’ parameter (now known as view). Rails’ #delegate method allows the passed method (#content_tag) to be implicitly called in the current context as if it was defined on the context’s ‘self’. The ‘to: :view’ is the object on which this method is found. For those unfamiliar with Rails, the content_tag creates a HTML tag whose type is that of the first parameter, returned as a string. Other tag options and enclosed tags can also be included as a hash param. If a block is given, anything within the block will be enclosed within the HTML tag.

Finally we get to the methods themselves.The first being the method called after the calendar instantiation in our helper method. Keep in mind, I included options hash containing styling classes from Bootstrap.

  ```ruby
  def table
    classes = "calendar table table-bordered"
    content_tag :table, class: classes do
      header + week_rows
    end
  end
  ```
  #table: adds a ‘table’ tag enclosing the return value of #header and #week_rows.

  ```ruby
  def header
    content_tag :tr, class:'bg-info' do
      HEADER.map { |day| content_tag :th, day, class:'text-center p-1' }.join.html_safe
    end
  end
  ```
  #header: returns a ‘tr’ (table row) wrapped around the joined new array from mapping over the HEADERS constant, and wrapping the day of the week in a ‘th’ (table header) tag.

  ```ruby
  def weeks
    first = date.beginning_of_month.beginning_of_week(START_DAY)
    last = date.end_of_month.end_of_week(START_DAY)
    (first..last).to_a.in_groups_of(7)
  end
  ```
  #weeks: first and last are assigned with the very explicit Rails methods, using the START_DAY constant to change the default start day from Monday to Sunday. The return of which is a range, converted into an array of dates to be displayed in the calendar, split into groups of 7, for each week. A nested array.

  ```ruby
  def week_rows
    weeks.map do |week|
      content_tag :tr, class:'bg-light' do
        week.map { |day| day_cell(day) }.join.html_safe
      end
    end.join.html_safe
  end
  ```
  #week_rows: mapping over the #weeks nested array wraps each week into a ‘tr’ tag that encloses each array of 7 days stored in the week variable. Through each iteration, the week array is mapped over replacing each date with…..
  
  ```ruby
  def day_cell(day)
    content_tag :td, view.capture(day, &callback), class: day_classes(day)
  end
  ```
  #day_cell: a ‘td’ (table cell) tag for each day of the week. The contents of these day cells is where things get slightly confusing. Remember way back, the ‘self’ object that was passed to the #calendar helper that had those two methods? This second method is #capture. Normally it is used to assign a piece of a template from a block to a variable. The block that was passed to the calendar helper, currently the callback variable, is the piece of our template we are saving to pass to this ‘td’ day_cell. The other variable ‘day’ passed to this #capture method is the ‘date’ param that is accessible in our callback block from the show view.

  ```ruby
  <%= calendar @date do |date| %>
    <%= date.day %>
  <% end %>
  ```
  In our example, the only contents of our day_cell ‘td’ is ‘date.day’ or the date of the month. Various events can be added to the calendar by first grouping the exposed data by date, then for each ‘td’ day_cell, check if there is an event on that date.

  ```ruby
  def day_classes(day)
    classes = ['day-cell']
    classes << 'table-success' if day == Date.today
    classes << 'table-active' if day.month != date.month
    classes.join(' ')
  end
  ```
  #day_classes: this method will add classes to each individual 'td'. classes can be added conditionally, for example, days not included in the current month or todays date.

##### Some of My Code

In my application, bills and chores are grouped by their due_dates.

```ruby
@date = params[:date] ? Date.parse(params[:date]) : Date.today
@bills = current_household.bills.group_by(&:due_date)
@chores = Chore.for(current_household)
@chores_by_date = @chores.group_by(&:due_date)
@events = @bills.merge(@chores_by_date)
```

If the day contained a bill or chore, the cell became a link to open a modal providing more information on the event.

```ruby
  <% if @events[date] %>
        <%= link_to date.day,
              household_calendar_day_path(current_household, date:date),
              remote:true,
              class:'btn btn-sm btn-block text-left font-blk' %>
        <%= content_tag :div,
              id:'calendar_bills',
              class:'small bg-danger text-light pl-1 pr-1' do %>
          <%= display_bills(@bills, date) %>
        <% end %>
        <%= content_tag :div,
              id:'calendar_chores',
              class:'small bg-warning pl-1 pr-1' do %>
          <%= display_chore(@chores_by_date, date) %>
        <% end %>
      <% else %>
        <%= date.day %>
      <% end %>
```

Using helper methods, paid and overdue bills were labeled accordingly.

```ruby
def paid_or_overdue(bill)
  if bill.paid?
    content_tag :span, class:'small text-success' do
      "- Paid!"
    end
  elsif bill.over_due?
    content_tag :span, class:'small text-danger' do
      "- Over Due!"
    end
  end
end
```

#### Styling

In its most basic state, the calendar is nothing more than a small table with  month, numbers and a couple links:

![Image of Calendar without CSS](/img/basic-calendar.jpeg)

With a little CSS and bootstrap classes, the results can be far more versatile:

![Image of Calendar with CSS/Bootstrap](/img/styled-calendar.jpeg)

##### Custom CSS
```css
.day-cell {
  height:100px !important;
  width:120px !important;
  padding:0 !important;
  vertical-align: top !important;
}

td {
  font-size: .85em;
  vertical-align: middle !important;
}
```

#### Conclusion

When I created this project, the aesthetics and ease of manipulation were the key reason I chose this design. Back then, it was a copy, paste and tweak job I promised myself I would later understand completely. This post has been a wonderful learning experience and fulfillment of a promise to myself.