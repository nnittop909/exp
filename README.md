rails g model product name released_on:date price:decimal

t.decimal :price, :precision => 2, :scale => 8

rails db:migrate

Copy the seed data to seeds.rb. 

rails db:seed

rails g controller products

def index
  @products = Product.order(:name)
end

Rails.application.routes.draw do
  resources :products
  
  root to: 'products#index'
end

Create layout.css with css shown in the github code download link. Add:

table td, table th {
  padding-right: 20px;
  padding-bottom: 5px;
  text-align: left;
  &:last-child {
    text-align: right;
  }
}

to products.scss. Create index.html.erb:

<h1>Products</h1>

<table id="products">
  <tr>
    <th>Product Name</th>
    <th>Release Date</th>
    <th>Price</th>
  </tr>

<%= render @products %>

</table>

Create product partial, `_product.html.erb`:

Replace the body section of the layout file:

<body>
  <div id="container">
    <% flash.each do |name, msg| %>
      <%= content_tag :div, msg, id: "flash_#{name}" %>
    <% end %>
    <%= yield %>
  </div>
</body>

You will now be able to view list of products in the home page.

Add download link to CSV and Excel in products index page.

<p>
  Download:
  <%= link_to "CSV", products_path(format: "csv") %> |
  <%= link_to "Excel", products_path(format: "xls") %>
</p>

Change the index action.

respond_to do |format|
  format.html
  format.csv { send_data @products.to_csv }
  format.xls 
end

Add the converter method to product model.

def self.to_csv(options = {})
  CSV.generate(options) do |csv|
    csv << column_names
    all.each do |product|
      csv << product.attributes
    end
  end
end

Click on the download link.

To respond to a custom format, register it as a MIME type first: http://guides.rubyonrails.org/action_controller_overview.html#restful-downloads. If you meant to respond to a variant like :tablet or :phone, not a custom format, be sure to nest your variant response within a format response: format.html { |html| html.tablet { ... } }

config/initializers/mime_types.rb

Mime::Type.register "application/xls", :xls

Restart the server. Click on download link again.

uninitialized constant Product::CSV

require 'csv'

in product model.

In rails console.

Product.column_names
 => ["id", "name", "released_on", "price", "created_at", "updated_at"]

rails c
Loading development environment (Rails 5.0.0.rc1)
> p = Product.first
  Product Load (0.1ms)  SELECT  "products".* FROM "products" ORDER BY "products"."id" ASC LIMIT ?  [["LIMIT", 1]]
 => #<Product id: 1, name: "Settlers of Catan", released_on: "2016-04-15", price: #<BigDecimal:7fada51e1018,'0.35E2',9(18)>, created_at: "2016-06-09 20:16:29", updated_at: "2016-06-09 20:16:29">
> p.attributes
 => {"id"=>1, "name"=>"Settlers of Catan", "released_on"=>Fri, 15 Apr 2016, "price"=>#<BigDecimal:7fada51e1018,'0.35E2',9(18)>, "created_at"=>Thu, 09 Jun 2016 20:16:29 UTC +00:00, "updated_at"=>Thu, 09 Jun 2016 20:16:29 UTC +00:00}
> p.attributes.values
 => [1, "Settlers of Catan", Fri, 15 Apr 2016, #<BigDecimal:7fada51e1018,'0.35E2',9(18)>, Thu, 09 Jun 2016 20:16:29 UTC +00:00, Thu, 09 Jun 2016 20:16:29 UTC +00:00]
> 

def self.to_csv(options = {})
  CSV.generate(options) do |csv|
    csv << column_names
    all.each do |product|
      p product
      csv << product.attributes.values
    end
  end
end

Since we are getting all the columns for this product, we retrieve the values for all the attributes of a given product. You will now be able to download the csv file.

Let's say you want to filter out created_at and updated_at fields, we can do this:

def self.to_csv(options = {})
  desired_columns = ["id", "name", "released_on", "price"]
  CSV.generate(options) do |csv|
    csv << desired_columns
    all.each do |product|
      csv << product.attributes.values_at(*desired_columns)
    end
  end
end

Create index.xls.erb with:

<?xml version="1.0"?>
<Workbook xmlns="urn:schemas-microsoft-com:office:spreadsheet"
  xmlns:o="urn:schemas-microsoft-com:office:office"
  xmlns:x="urn:schemas-microsoft-com:office:excel"
  xmlns:ss="urn:schemas-microsoft-com:office:spreadsheet"
  xmlns:html="http://www.w3.org/TR/REC-html40">
  <Worksheet ss:Name="Sheet1">
    <Table>
      <Row>
        <Cell><Data ss:Type="String">ID</Data></Cell>
        <Cell><Data ss:Type="String">Name</Data></Cell>
        <Cell><Data ss:Type="String">Release Date</Data></Cell>
        <Cell><Data ss:Type="String">Price</Data></Cell>
      </Row>
    <% @products.each do |product| %>
      <Row>
        <Cell><Data ss:Type="Number"><%= product.id %></Data></Cell>
        <Cell><Data ss:Type="String"><%= product.name %></Data></Cell>
        <Cell><Data ss:Type="String"><%= product.released_on %></Data></Cell>
        <Cell><Data ss:Type="Number"><%= product.price %></Data></Cell>
      </Row>
    <% end %>
    </Table>
  </Worksheet>
</Workbook>

Click on Excel download link. You will be able to download and view the excel spreadsheet.




