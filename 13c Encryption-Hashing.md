# Encryption

## Introduction

Laravel's encrypter uses `OpenSSL` to provide `AES-256` and `AES-128` encryption. All of Laravel's encrypted values are signed using a message authentication code (MAC) so that their underlying value can not be modified once encrypted.
                                                                            
## Configuration

Set a `key` option in `config/app.php` configuration file. You should use the `php artisan key:generate` command to generate this key.

## Using The Encrypter

### Encrypting A Value

* Encrypt a value using the `encrypt` helper.
* All encrypted values are encrypted using OpenSSL and the AES-256-CBC cipher.
* All encrypted values are signed with a message authentication code (MAC) to detect any modifications to the encrypted string.

```php
public function storeSecret(Request $request, $id)
{
    $user = User::findOrFail($id);

    $user->fill([
        'secret' => encrypt($request->secret)
    ])->save();
}
```

### Encrypting Without Serialization

Non-PHP clients receiving encrypted values will need to unserialize the data.

```php
use Illuminate\Support\Facades\Crypt;

$encrypted = Crypt::encryptString('Hello world.');

$decrypted = Crypt::decryptString($encrypted);
```

### Decrypting A Value

If the value can not be properly decrypted, such as when the MAC is invalid, an `Illuminate\Contracts\Encryption\DecryptException` will be thrown

```php
use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}
```

# Hashing

## Introduction

The Laravel `Hash` facade provides secure **Bcrypt** hashing for storing user passwords.

## Basic Usage

```php
public function update(Request $request)
{
    // Validate the new password length...

    $request->user()->fill([
        'password' => Hash::make($request->newPassword)
    ])->save();
}
```

The `make` method also allows you to manage the work factor of the `bcrypt` hashing algorithm using the `rounds` option; however, the default is acceptable for most applications.

```php
$hashed = Hash::make('password', [
    'rounds' => 12
]);
```

### Verifying A Password Against A Hash

The `check` method allows you to verify that a given plain-text string corresponds to a given hash.

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}
```

### Checking If A Password Needs To Be Rehashed

The needsRehash function allows you to determine if the work factor used by the hasher has changed since the password was hashed.

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

