---
layout: post
title:  "Import, export CSV with Ruby (part 2)"
date:   2015-07-28 23:00:00
categories: rails csv
---

I'm glad to see you again.

The importing part is a little longer than the [exporting part](/rails/csv/2015/07/18/import-export-csv-with-ruby-part-1.html) but I guarantee it's still very simple!

# Say hi to Import

## Route

{% highlight ruby %}
  resources :csv_imports, only: [:create]
{% endhighlight %}

## Controller

{% highlight ruby %}
class CsvImportsController < ApplicationController
  before_action :check_params, only: :create

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
  def check_params
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

{% highlight ruby %}
# trigger button
<%= link_to "Import", "#", class: %w(btn search-form-toggle), data: {target: "import-form"} %>
{% endhighlight %}

{% highlight ruby %}
# _import_form.html.erb
<%= form_tag csv_imports_path, method: :post, multipart: true, id: "import-form", class: "search-form form-inline" do %>
  <div>
    <%= label_tag :file, "CSV: " %>
    <%= file_field_tag :file %>
    <%= hidden_field_tag :model, model %>
    <%= submit_tag "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
{% endhighlight %}


Here, ```headers: true``` means it treats first row as the CSV's header. When the CSV file has header you can easily access the content by the header, eg: ```row_hash["id"]```, where id is the first column of the CSV file.

The CSV file contain the id column so the ```import_csv``` will update the record if it do exist. Otherwise, it will create a brand new record.

In the ```import_csv``` method I use the bang method like ```update_attributes!``` and ```create!``` so it will raise the error if there's any problem with the updating and creating record.

Then I puts the ```import_csv``` into a transaction so tt'll create/update all records or do nothing (rollback) if any error is raised inside the transaction, and we rescue the error by redirecting back with a flash message.

If you'd like to know the exact error which make the transaction rolled back, you can ```rescue => e```, then ```puts e``` to get the detail error or use any debug tool such as [byebug](https://github.com/deivid-rodriguez/byebug), [pry-rails](https://github.com/rweng/pry-rails).


Hope it's useful!
