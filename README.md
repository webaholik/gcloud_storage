# GcloudStorage

Simple Google Cloud Storage file upload gem for Ruby. This is an alternative gem
for carrierwave with fog. As, carrierwave with fog only uses API Key
authentication to talk to Google Cloud Storage API. This gem supports the
service account authentication and as well as compute instance service account
where you don't have to initialize the gem with the credentials.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'gcloud_storage'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install gcloud_storage

## Usage
Mountable ActiveRecord model configuration looks like this.

```ruby
class TempFile < ActiveRecord::Base
  include GcloudStorage::Uploader

  # attribute :file
  mount_gcloud_uploader :file
end

temp_file = TempFile.new(file_uploader_object: path_to_file) # => TempFile object
temp_file.file_url # => HTTPS URL
temp_file.file_path # => HTTPS URL
```

Create an initializer file `config/initializers/gcloud_storage.rb` and add these
lines:

```ruby
# Uncomment this to support large file uploads
# Faraday.default_adapter = :httpclient

GcloudStorage.configure do |config|
  config.credentials = {
    bucket_name: 'bucket_name', # Storage bucket name
    project_id: 'project_id',   # Google Cloud Project ID
    key_file: 'key_file_path'   # Compute Service account json file
  }
end

# Add this to validate and cache connection object
# while loading the Rails Application
# GcloudStorage.initialize_service!

```

File upload example:
You can pass path to the file to be uploaded as a `String` or as `Pathname` or
as `Rack::Multipart::UploadedFile` object using HTML Multipart form.
The attribute to initialize will be `#{column}_uploader_object`. Here the column
name is file in the above example.

```
$ rails console
Loading development environment (Rails 4.2.0)
 :001 > `echo "This is a test file" > temp.txt`
 => ""
 :002 > file = Rack::Multipart::UploadedFile.new("temp.txt")
 => #<Rack::Multipart::UploadedFile:0x007f9e29ae37c8 @content_type="text/plain", @original_filename="temp.txt", @tempfile=#<Tempfile:/var/folders/temp.txt>>
 :003 > temp_file = TempFile.new(file_uploader_object: file) # file_attribute => #{column}_uploader_object
 => #<TempFile id: nil, file: "temp.txt", created_at: nil, updated_at: nil>
 :004 > temp_file.valid?
 => true
 :005 > temp_file.save
 => true
 :006 > temp_file.file_url
 => "https://storage.googleapis.com/<bucket-name>/uploads/temp_files/1/temp.txt?GoogleAccessId=compute%40developer.gserviceaccount.com&Expires=1459851006&Signature=XXXX"
 :007 > `echo "Yet Another test file" > tmp/yet_another_test.txt`
 => ""
 :008 > another_file = TempFile.new(file_uploader_object: "tmp/yet_another_test.txt")
 => #<TempFile id: nil, file: "yet_another_test.txt", created_at: nil, updated_at: nil>
 :009 > another_file.save
 => true
 :010 > another_file.file_url
 => "https://storage.googleapis.com/<bucket-name>/uploads/temp_files/2/yet_another_test.txt?GoogleAccessId=compute%40developer.gserviceaccount.com&Expires=1459851800&Signature=XXXX"
 :011 > open(another_file.file_url).read
 => "Yet Another test file\n"
 :012 > another_file.file_path
 => "uploads/temp_files/2/yet_another_test.txt"
```

## TODO
1. Write specs
2. Support multiple mountable columns
3. Support overriding the mountable methods in a better way. For now the user
   can override the methods in the model.

## Development

After checking out the repo, run `bin/setup` to install dependencies. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/itsprdp/gcloud_storage. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
