---
layout: post
title:  "Import, export CSV with Ruby"
date:   2015-07-17 23:22:00
categories: rails csv
---

Import, export CSV is one of the most popular feature that we usually need for our project.

It's very simple and I'll show you how to make it just with Ruby CSV library.

# Say hi to Export

## route.rb

{% highlight ruby %}
  resources :csv_imports, only: [:create]
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

{% highlight ruby %}
# _export_form.html.erb
<%= form_tag csv_exports_path, method: :post, multipart: true, class: "search-form" do %>
  <%= hidden_field_tag :model, model %>
  <% @search = model.search params[:q] %>
  <%= hidden_field_tag :object_ids, @search.result.ids %>

  <div class="text-center">
    <%= submit_tag I18n.t("views.buttons.export"), class: "btn btn-primary" %>
  </div>
<% end %>

{% endhighlight %}

Here, I'm using ransack as object searching so you can see:

{% highlight ruby %}
<% @search = model.search params[:q] %>
{% endhighlight %}

It just exports records of search result. If you want to export all exist records in database, you can remove all the codes that relate to ```object_ids``` in this post.

I recommend to use this line of code for importing purpose:

{% highlight ruby %}
exceptions = %w( created_at updated_at ) # if you don't need to export timestamps
{% endhighlight %}

Because when you import, it means that you create new record and you don't need to use the timestamps in csv file anymore.

# Say hi to Import

It's easy, right? See you next week with Importing part.
