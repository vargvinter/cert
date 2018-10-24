# Configuration/Drivers

The filesystem configuration file is located at config/filesystems.php. Within this file you may configure all of your "disks". Each disk represents a particular storage driver and storage location. 

## The Public Disk

* By default, the  `public` disk uses the `local` driver and stores these files in `storage/app/public`.
* To make them accessible from the web, you create a symbolic link from `public/storage` to  `storage/app/public`.
* `php artisan storage:link`.

## The Local Driver

* When using the `local` driver, all file operations are relative to the `root` directory.
* By default, this value is set to the `storage/app` directory.

```php
Storage::disk('local')->put('file.txt', 'Contents');
```

## Other drivers/disks

* S3 Driver.
* FTP Driver.
* Rackspace Driver.

# Storing / Retrieving Files

## Retrieving Files

```php
$contents = Storage::get('file.jpg');
```

```php
$exists = Storage::disk('s3')->exists('file.jpg');
```

### File URLs

You may use the url method to get the URL for the given file. If you are using the local driver, this will typically just prepend /storage to the given path and return a relative URL to the file. If you are using the s3 or rackspace driver, the fully qualified remote URL will be returned:

```php
$url = Storage::url('file1.jpg');
```

### Temporary URLs

For files stored using the s3 or rackspace driver, you may create a temporary URL.

```php
$url = Storage::temporaryUrl(
    'file1.jpg', now()->addMinutes(5)
);
```

### File Metadata

```php
$size = Storage::size('file1.jpg');

$time = Storage::lastModified('file1.jpg');
```

## Storing Files

* The `put` method may be used to store raw file contents on a disk.
* You may also pass a PHP  `resource` to the `put` method, which will use Flysystem's underlying stream support.

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

### Automatic Streaming

* Use the putFile or putFileAs method.
* This method accepts either a  `Illuminate\Http\File` or `Illuminate\Http\UploadedFile` instance and will automatically stream the file to your desired location.

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// Automatically generate a unique ID for file name...
Storage::putFile('photos', new File('/path/to/photo'));

// Manually specify a file name...
Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

* we only specified a directory name, not a file name.
* By default, the putFile method will generate a unique ID to serve as the file name.
* The path to the file will be returned by the putFile method so you can store the path, including the generated file name, in your database.
* The putFile and putFileAs methods also accept an argument to specify the "visibility" of the stored file.
* This is particularly useful if you are storing the file on a cloud disk such as S3.

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

### Prepending & Appending To Files

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

### Copying & Moving Files

```php
Storage::copy('old/file1.jpg', 'new/file1.jpg');

Storage::move('old/file1.jpg', 'new/file1.jpg');
```

## File Uploads

In web applications, one of the most common use-cases for storing files is storing user uploaded files.

```php
class UserAvatarController extends Controller
{
    /**
     * Update the avatar for the user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

You may also call the putFile method on the Storage facade to perform the same file manipulation as the example above:

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

### Specifying A File Name

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

### Specifying A Disk

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

# Custom Filesystems

In order to set up the custom filesystem you will need a Flysystem adapter. Let's add a community maintained Dropbox adapter to our project.

`composer require spatie/flysystem-dropbox`

```php
public function boot()
{
    Storage::extend('dropbox', function ($app, $config) {
        $client = new DropboxClient(
            $config['authorizationToken']
        );

        return new Filesystem(new DropboxAdapter($client));
    });
}
```

