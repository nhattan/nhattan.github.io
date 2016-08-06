---
layout: post
title:  "Import, export CSV with Ruby (part 2)"
date:   2015-07-28 23:00:00
categories: rails csv
---

Hey, so glad to see you again!

This post is a little bit longer than the [Part 1](/rails/csv/2015/07/18/import-export-csv-with-ruby-part-1.html) but still very simple and it just takes you about 10 minutes.

# Import

## Route

{% highlight ruby %}
  resource :csv_import, only: [:create]
{% endhighlight %}

## Controller

{% highlight ruby %}
class CsvImportsController < ApplicationController
  before_action :validate_model_and_file, only: :create

  def create
    begin
      ActiveRecord::Base.transaction do
        import_csv @model, @file
      end
      redirect_to :back, notice: "Import successfully"
    rescue
      redirect_to :back, alert: "Import failed"
    end
  end

  private
  def validate_model_and_file
    @model = params[:model].safe_constantize
    @file = params[:file]
    unless @model && @file.present?
      redirect_to :back, alert: "Import failed"
    end
  end

  def import_csv model, file
    exceptions = %w( created_at updated_at )
    CSV.foreach(file.path, headers: true) do |row|
      row_hash = row.to_hash.except(*exceptions)
      if record = model.find_by(id: row_hash["id"])
        record.update_attributes! row_hash
      else
        model.create! row_hash
      end
    end
  end
end
{% endhighlight %}

## View

{% highlight erb %}
# import button
<%= link_to "Import", "#", class: %w(btn search-form-toggle), data: {target: "import-form"} %>
{% endhighlight %}

{% highlight erb %}
# _import_form.html.erb
<%= form_tag csv_import_path, method: :post, multipart: true, id: "import-form", class: "search-form form-inline" do %>
  <div>
    <%= label_tag :file, "CSV: " %>
    <%= file_field_tag :file %>
    <%= hidden_field_tag :model, model %>
    <%= submit_tag "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
{% endhighlight %}


```headers: true``` means it treats the first row as the CSV's header. If the CSV file has header you can easily read the content by the header, eg: ```row_hash["id"]```, where ```id``` is the first column of the CSV file.

If the CSV file contains the ```id``` column, ```import_csv``` will update the record if it do exist. Otherwise, it will create a brand new record.

In ```import_csv``` I use the bang method ```update_attributes!``` and ```create!```, it will raise the error if there's any problem in updating and creating record.

I put ```import_csv``` into a transaction so it'll create/update all records or do nothing (rollback) if any error is raised inside the transaction, and we can rescue the error by redirecting back with a flash message.

If you'd like to see what error was raised inside the transaction, you can use ```rescue => e``` instead of ```rescue``` in the controller, then ```puts e``` to get the error detail or use any debug tool such as [byebug](https://github.com/deivid-rodriguez/byebug) or [pry-rails](https://github.com/rweng/pry-rails).

Hope this helps. Peace out!