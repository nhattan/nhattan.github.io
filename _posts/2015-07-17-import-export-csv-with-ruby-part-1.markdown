---
layout: post
title:  "Import, export CSV with Ruby (part 1)"
date:   2015-07-17 23:22:00
categories: rails csv
---

CSV is a common data exchange format that is widely supported in many applications. And import, export CSV is one of the most common features in applications.

It's so simple to build this feature by using Ruby CSV library in a Ruby on Rails application.

In this post, I'll show you how to import records from CSV and export records to CSV on any resources.

# Export

## Route

{% highlight ruby %}
  resource :csv_export, only: [:create]
{% endhighlight %}

## Controller

{% highlight ruby %}
class CsvExportsController < ApplicationController
  before_action :check_params, only: :create

  def create
    begin
      timestamp = Time.current.strftime("%Y-%m-%d-%H-%M-%S")
      filename = "#{@model.table_name}-#{timestamp}.csv"
      send_data export_csv(@model, @object_ids), filename: filename
    rescue
      redirect_to :back, alert: "Export failed"
    end
  end

  private
  def check_params
    @model = model.safe_constantize # return nil if the model is not exist
    unless @model
      redirect_to :back, alert: "Export failed"
    end
  end

  def export_csv model, object_ids
    objects = object_ids.present? ? model.where(id: object_ids) : model.all
    exceptions = %w( created_at updated_at ) # if you don't need to export timestamps
    attrs = model.column_names.reject{ |c| exceptions.include?(c) }
    CSV.generate do |csv|
      csv << attrs
      objects.each do |object|
        csv << object.attributes.values_at(*attrs)
      end
    end
  end
end
{% endhighlight %}

## View

{% highlight erb %}
# export button
<%= link_to "Export", "#", class: %w(btn search-form-toggle), data: {target: "export-form"} %>
{% endhighlight %}

{% highlight javascript %}
// js for search-form-toggle
$(function() {
  return $(".search-form-toggle").click(function(e) {
    var target;
    e.preventDefault();
    target = $(this).attr("data-target");
    return $("#" + target).slideToggle("fast");
  });
});
{% endhighlight %}

{% highlight erb %}
# _export_form.html.erb
<%= form_tag csv_export_path, method: :post, multipart: true, id: "export-form", class: "search-form" do %>
  <%= hidden_field_tag :model, model %>
  <% @search = model.search params[:q] %>
  <%= hidden_field_tag :object_ids, @search.result.ids %>

  <div class="text-center">
    <%= submit_tag "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
{% endhighlight %}

As you can see, I'm using [ransack](https://github.com/activerecord-hackery/ransack) as object searching:

{% highlight erb %}
<% @search = model.search params[:q] %>
{% endhighlight %}

It just exports records of search result. If you want to export all exist records in database, remove all the codes that relate to ```object_ids``` in this post.

I recommend using this line of code for importing:

{% highlight ruby %}
exceptions = %w( created_at updated_at ) # if you don't need to export timestamps
{% endhighlight %}

Because when you import, it means that you create new record and you don't need to use the timestamps in CSV file anymore.


So we've done with exporting records to CSV. It's so easy, right? I told you :). You also can see the snippet used to export records here, it's pretty simple [https://gorails.com/episodes/export-to-csv](https://gorails.com/episodes/export-to-csv)

Let's go to part 2:
[Import, export CSV with Ruby (part 2)](/rails/csv/2015/07/29/import-export-csv-with-ruby-part-2.html)
