
Connecting to DynamoDB 

c = Aws::DynamoDB::Client.new(endpoint: "http://localhost:8000",region: "localhost")

List the Tables

c.list_tables

Aws::Errors::MissingCredentialsError: unable to sign request without credentials set. Provide credentials to fix the problem.

Aws.config.update({
  region: 'localhost',
  credentials: Aws::Credentials.new('puff', 'daddy'),
  endpoint:'http://localhost:8000'
})

=> {:region=>"localhost", :credentials=>#<Aws::Credentials access_key_id="puff">, :endpoint=>"http://localhost:8000"} 
> c = Aws::DynamoDB::Client.new(endpoint: "http://localhost:8000",region: "localhost")
 => #<Aws::DynamoDB::Client> 
> c.list_tables
 => #<struct Aws::DynamoDB::Types::ListTablesOutput table_names=["Image", "ImageTag", "Tag"], last_evaluated_table_name=nil>

Scan the Tag Table
   
c.scan({table_name: 'Tag'})

resp.items #=> Array
resp.items[0] #=> Hash
resp.items[0]["AttributeName"] #=> <Hash,Array,String,Numeric,Boolean,IO,Set,nil>
resp.count #=> Integer
resp.scanned_count #=> Integer

The scanned_count is the number of items evaluated, before any ScanFilter is applied.

resp.items[0]
 => {"ImageCount"=>0.1e1, "Tag"=>"Amazon VPC"} 
 > resp.items[0]['ImageCount']
 => 0.1e1 
 > resp.items[0]['Tag']
 => "Amazon VPC" 
 
Getting Information about All Tables

dynamoDB = Aws::DynamoDB::Resource.new(region: 'localhost')
      
dynamoDB.tables.each do |t|
  puts "Name:    #{t.name}"
  puts "#Items:  #{t.item_count}"
end
 
Creating a Simple Table with a Single Primary Key

attribute_defs = [
  { attribute_name: 'ID',        attribute_type: 'N' },
  { attribute_name: 'FirstName', attribute_type: 'S' },
  { attribute_name: 'LastName',  attribute_type: 'S' }
]

key_schema = [
  { attribute_name: 'ID', key_type: 'HASH' }
]

index_schema = [
  { attribute_name: 'FirstName', key_type: 'HASH'  },
  { attribute_name: 'LastName',  key_type: 'RANGE' }
]

global_indexes = [{
  index_name:             'LastNameFirstNameIndex',
  key_schema:             index_schema,
  projection:             { projection_type: 'ALL' },
  provisioned_throughput: { read_capacity_units: 5, write_capacity_units: 10 }
}]

request = {
  attribute_definitions:    attribute_defs,
  table_name:               'Users',
  key_schema:               key_schema,
  global_secondary_indexes: global_indexes,
  provisioned_throughput:   { read_capacity_units: 5, write_capacity_units: 10 }
}

dynamodb_client = Aws::DynamoDB::Client.new(region: 'localhost')
dynamodb_client.create_table(request)
dynamodb_client.wait_until(:table_exists, table_name: 'Users')

Adding an Item to a Table

dynamoDB = Aws::DynamoDB::Resource.new(region: 'localhost')

table = dynamoDB.table('Users')

table.put_item({
  item:
    {
      "ID" => 123456,
      "FirstName" => 'Snoop',
      "LastName" => 'Doug'
}})
 
Getting Information about the Items in a Table


dynamoDB = Aws::DynamoDB::Resource.new(region: 'localhost')

table = dynamoDB.table('Tag')

scan_output = table.scan({
  limit: 50,
  select: "ALL_ATTRIBUTES"
})

scan_output.items.each do |item|
  keys = item.keys

  keys.each do |k|
    puts "#{k}: #{item[k]}"
  end
end

Getting Information about a Specific Item in a Table

dynamoDB = Aws::DynamoDB::Resource.new(region: 'localhost')

table = dynamoDB.table('Users')

resp = table.get_item({
  key: { 'ID' => 123456 }
})

first_name = resp.item['FirstName']
last_name = resp.item['LastName']
         
puts "First name: #{first_name}"
puts "Last name:  #{last_name}"

Updating a Table

dynamoDB = Aws::DynamoDB::Resource.new(region: 'localhost')

table = dynamoDB.table('Users')

Get the IDs of all of the users
resp = table.scan({ select: "ALL_ATTRIBUTES" })

resp.items.each do |item|
  id = item['ID']
  
  request = {
    key: { 'ID' => id },
    update_expression: 'set airmiles=:pVal',
    expression_attribute_values: { ':pVal' => '10000' }
  }

  # Update the item in the table:
  table.update_item(request)
end

References

- [Amazon DynamoDB Examples](http://docs.aws.amazon.com/sdk-for-ruby/v2/developer-guide/dynamo-examples.html)
- [Ruby on Rails with Cloud AWS Dynamodb](http://codepany.com/blog/ruby-on-rails-with-local-and-cloud-aws-dynamodb/)
- [AWS SDK for Ruby DynamoDB Docs](http://docs.aws.amazon.com/sdkforruby/api/Aws/DynamoDB/Client.html#scan-instance_method)
- [Step by Step Guid to Using DynamoDB with Rails](https://blog.faodailtechnology.com/step-by-step-guide-to-using-dynamodb-with-rails-application-i-a676cb9ba4df)