## file_fixture_upload

You can use `fixture_file_upload` in tests to imitiate an uploaded file(ActionDispatch::Http::UploadedFile).

Keep in mind that this does not close the file, so make sure to close after your tests are finished.

```ruby
@file = file_fixture_upload(path, mime_type, binary = false)

after do 
  @file.close
end
```
