# libgenaro-android

Asynchronous Android library and CLI for encrypted file transfer on the Genaro network written in Java 8. It's compatible with [libgenaro](https://github.com/GenaroNetwork/libgenaro).

## Features

- Load wallet from json file with password
- Use wallet to sign request for user authentication
- Delete bucket/rename bucket/list buckets/list files/delete file/upload file/download file
- Asynchronous I/O with concurrent peer-to-peer network requests for shards
- Erasure encoding with reed solomon for data reliability
- File encryption with AES-256-CTR
- File name and bucket name encryption with AES-256-GCM
- Asynchronous progress updates
- Retry several times when upload/download is failed
- Seed based file encryption key for portability between devices
- File integrity and authenticity verified with HMAC-SHA512
- Proxy support
- Exchange report with bridge
- File encryption key can be provided to decrypt encrypted file
- Command line interface
- Mock bridge and farmer, and continous integration
- String literal can be encrypted with AES-256-CTR and directly stored to a bucket

## Issues

- Upload or download file of large size(>256MB) will not use Reed-Solomon algorithm, or it may cause an OutOfMemoryError, becasue the the JavaReedSolomon library doesn't support memory mapped files.

## 3rd party dependencies

- [Spongy Castle](https://rtyley.github.io/spongycastle/) for crypto algorithms.
- [dnsjava](http://www.xbill.org/dnsjava/) for base16 encoding.
- [web3j](https://web3j.io/) for wallet managment, BIP39 and Interaction with blockchain.
- [JavaReedSolomon](https://github.com/Backblaze/JavaReedSolomon) for reed solomon algorithm.
- [jackson](https://github.com/FasterXML/jackson) for JSON parse/compose.
- [okhttp3](https://github.com/square/okhttp) as HTTP client.
- [java-getopt](https://www.urbanophile.com/arenn/hacking/download.html) a Java command line option parser that is compatible with GNU getopt.
- [guava](https://github.com/google/guava) and [apache.commons](https://commons.apache.org/) as utility.
- [testng](https://testng.org/doc/index.html) for testing.

## Used as 3rd party package

1. Import libgenaro-android as a module of your app

2. Add libgenaro-android as a dependency of your app, something like: 

```xml
dependencies {
    implementation project(':libgenaro-android')
}
```

3. Add repository for dnsjava to build.gradle of your app:

```gradle
repositories {
    maven { url "http://www.jabylon.org/maven/" }
}
```

4. Add READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE and INTERNET permissions to AndroidManifest.xml of your app:

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.INTERNET" />
```

5. Your app should request for READ_EXTERNAL_STORAGE and WRITE_EXTERNAL_STORAGE permission before upload and download, such as:

```java
ActivityCompat.requestPermissions(this, new String[] {Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE}, MY_REQUEST_CODE);
```

(PS: the JDK version of your app must be 1.8 or higher)

## APIs

```java
/**
 * @brief Get Genaro bridge API information.
 *
 * @return The Genaro bridge API information.
 */
public String getInfo()

/**
 * @brief List available buckets for a user.
 *
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> getBuckets(final GetBucketsCallback callback)

/**
 * @brief Delete a bucket.
 *
 * @param[in] bucketId The bucket id
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> deleteBucket(final String bucketId, final DeleteBucketCallback callback)

/**
 * @brief Rename a bucket.
 *
 * @param[in] bucketId The bucket id
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> renameBucket(final String bucketId, final String name, final RenameBucketCallback callback)

/**
 * @brief Get a list of all files in a bucket.
 *
 * @param[in] bucketId The bucket id
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> listFiles(final String bucketId, final ListFilesCallback callback)

/**
 * @brief Get mirror data for a file.
 *
 * @param[in] bucketId The bucket id
 * @param[in] fileId The file id
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> listMirrors(final String bucketId, final String fileId, final ListMirrorsCallback callback)

/**
 * @brief Delete a file in a bucket.
 *
 * @param[in] bucketId The bucket id
 * @param[in] fileId The file id
 * @param[in] callback The callback when complete
 * @return A CompletableFuture.
 */
public CompletableFuture<Void> deleteFile(final String bucketId, final String fileId, final DeleteFileCallback callback)

/**
 * @brief Download a file
 *
 * @param[in] bucketId The bucket id
 * @param[in] fileId The file id
 * @param[in] filePath The file path
 * @param[in] overwrite Whether to overwrite if exists
 * @param[in] key The key of AES for decryption
 * @param[in] ctr The ctr of AES for decryption
 * @param[in] isDecrypt Whether to decrypt the downloaded data
 * @param[in] callback The callback on progress or when complete
 * @return A Downloader.
 */
public Downloader resolveFile(final String bucketId, final String fileId, final String filePath, final boolean overwrite, final boolean isDecrypt, final String keyBase16, final String ctrBase16, final ResolveFileCallback callback) throws GenaroException

/**
 * @brief Upload a file
 *
 * @param[in] rs Whether to use Reed-Solomon to generate parity shards
 * @param[in] fileOrData The file path or the text
 * @param[in] isFilePath Whether fileOrData is file path or text
 * @param[in] fileName The file name
 * @param[in] bucketId The bucket id
 * @param[in] ei The encryption info for file encryption and decryption(can be generated by function generateEncryptionInfo)
 * @param[in] callback The callback on progress or when complete
 * @return A Uploader.
 */
public Uploader storeFile(final boolean rs, final String filePath, final String fileName, final String bucketId, EncryptionInfo ei, final StoreFileCallback callback) throws GenaroException

/**
 * @brief Decrypt a file
 *
 * @param[in] filePath The undecrypted file path
 * @param[in] key The key of AES
 * @param[in] iv The ctr of AES
 * @return The decrypted text.
 */
public static String decryptFileToText(String filePath, byte[] key, byte[] iv) throws GenaroException

/**
 * @brief Encrypt the meta use AES-256-GCM combined with HMAC-SHA512 to filePath
 *
 * @param[in] meta The meta to be decrypted
 * @param[in] filePath The file path
 */
public void encryptMetaToFile(String meta, String filePath) throws GenaroException

/**
 * @brief Decrypt the meta in filePath use AES-256-GCM combined with HMAC-SHA512
 *
 * @param[in] filePath The file path
 * @return The decrypted meta.
 */
public String decryptMetaFromFile(String filePath) throws GenaroException
```

## Example Usage

Initialize:

```java
String bridgeUrl = "http://47.100.33.60:8080";
String V3JSON = "{ \"address\": \"aaad65391d2d2eafda9b2732000001d52a6a3dc8\",
        \"crypto\": { \"cipher\": \"aes-128-ctr\",
        \"ciphertext\": \"e968751f3d60827b6e6000006c024ecc82f33a6c55428be33249c83edba444ca\",
        \"cipherparams\": { \"iv\": \"e80d9ec9b00000a143c756ec78066ad9\" }, \"kdf\": \"scrypt\",
        \"kdfparams\": { \"dklen\": 32, \"n\": 262144, \"p\": 1, \"r\": 8, \"salt\":
        \"ea7cb2b004db67d000003790caced7a96b636762f280b243e794fb5bef8ef74b\" },
        \"mac\": \"ceb3789e77be8f2a7ab4d000001b54e048ad3f5b080b96e07759de7442e050d2\" },
        \"id\": \"e28f31b4-1f43-428b-9b12-ab58000004b1\", \"version\": 3 }";
String passwd = "xxxxxx";
Genaro api;
try {
    api = new Genaro(bridgeUrl, V3JSON, passwd);
} catch (Exception e) {
    return;
}
```

List buckets:

```java
CompletableFuture<Void> fu = api.getBuckets(new GetBucketsCallback() {
    @Override
    public void onFinish(Bucket[] buckets) { }
    @Override
    public void onFail(String error) { }
});

// getBuckets is Non-Blocking, if you want to wait until it is finished, call fu.join()
```

Delete bucket:

```java
String bucketId = "5bfcf77cea9b6322c5abd929";
CompletableFuture<Void> fu = api.deleteBucket(bucketId, new DeleteBucketCallback() {
    @Override
    public void onFinish() { }
    @Override
    public void onFail(String error) { }
}));

// deleteBucket is Non-Blocking, if you want to wait until it is finished, call fu.join()
```

Rename bucket:

```java
String bucketId = "5bfcf77cea9b6322c5abd929";
String newName = "abc";
CompletableFuture<Void> fu = api.renameBucket(bucketId, newName, new RenameBucketCallback() {
    @Override
    public void onFinish() { }
    @Override
    public void onFail(String error) { }
}

// renameBucket is Non-Blocking, if you want to wait until it is finished, call fu.join()
```

List files:

```java
String bucketId = "5bfcf77cea9b6322c5abd929";
CompletableFuture<Void> fu = api.listFiles(bucketId, new ListFilesCallback() {
    @Override
    public void onFinish(GenaroFile[] files) { }
    @Override
    public void onFail(String error) { }
}

// listFiles is Non-Blocking, if you want to wait until it is finished, call fu.join()
```

Delete file:

```java
String bucketId = "5bfcf77cea9b6322c5abd929";
String fileId = "5c0e1289bbdd6f2d157dd8b2";
CompletableFuture<Void> fu = api.deleteFile(bucketId, fileId, new DeleteFileCallback() {
    @Override
    public void onFinish() { }
    @Override
    public void onFail(String error) { }
}

// deleteFile is Non-Blocking, if you want to wait until it is finished, call fu.join()
```

Upload file:

```java
String bucketId = "5bfcf4ea7991d267f4eb53b4";
String fileOrData = "xxxxxx";
boolean isFilePath = true;
String fileName = "abc.txt";
boolean rs = true;

EncryptionInfo ei = genaro.generateEncryptionInfo(null, bucketId);
Uploader uploader = null;
try {
    uploader = api.storeFile(rs, fileOrData, isFilePath, fileName, bucketId, ei, new StoreFileCallback() {
        @Override
        public void onBegin(long fileSize) { }
        @Override
        public void onProgress(float progress) { }
        @Override
        public void onCancel() { }
        @Override
        public void onFail(String error) { }
        @Override
        public void onFinish(String fileId, byte[] sha256OfEncrypted) { }
    });
} catch (GenaroException e) { }

// storeFile is Non-Blocking, if you want to wait until it is finished, call uploader.join()
// if you want to cancel it, call uploader.cancel()
```

Download file:

```java
String bucketId = "5bfcf4ea7991d267f4eb53b4";
String fileId = "5c0103fd5a158a5612e67461";
String filePath = "xxxxxx";

Downloader downloader = null;
try {
    downloader = genaro.resolveFile(bucketId, fileId, path, true, true, null, null, new ResolveFileCallback() {
        @Override
        public void onBegin() { }
        @Override
        public void onProgress(float progress) { }
        @Override
        public void onCancel() { }
        @Override
        public void onFail(String error) { }
        @Override
        public void onFinish(long fileBytes, byte[] sha256) { }
    });
} catch (GenaroException e) { }

// resolveFile is Non-Blocking, if you want to wait until it is finished, call downloader.join()
// if you want to cancel it, call downloader.cancel()
```
