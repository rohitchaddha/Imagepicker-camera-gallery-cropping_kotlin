# Imagepicker-camera-gallery-cropping_kotlin
Android choosing Image from Camera / Gallery with Crop Functionality in Kotlin  
Taking pictures from camera or gallery is an essential feature for many applications those includes media in their apps. A simple notes app may need a profile picture to make the notes more personal. Getting a thumbnail image from the camera is easy, but sometimes you want the full resolution image without storing it in a gallery, crop it and avoid the possible memory exceptions.
Android sample project demonstrating choosing an image from gallery or camera with the cropping functionality in kotlin

uCrop
Thanks to Yalantis for providing such a beautiful cropping (uCrop) library. This example uses the uCrop library for cropping functionality.
How to Use
Follow the below simple steps to add the library into your project.
1.	Add file_provider.xml to your res -> xml foler.
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-cache-path
        name="cache"
        path="camera" />
</paths>
2.	In AndroidManifext.xml, add CAMERA, DEXTER and STORAGE permissions. Add UCrop activity and FileProvider paths.
3.	<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          package="com.app.imagepicker_cropping_kotlin">
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

    <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            tools:ignore="GoogleAppIndexingWarning"
            android:theme="@style/AppTheme">
        <activity android:name=".ImagePickerActivity" />
        <activity android:name=".MainActivity"
                  android:theme="@style/AppTheme.NoActionBar"
                  android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        <!-- uCrop cropping activity -->
        <activity
                android:name="com.yalantis.ucrop.UCropActivity"
                android:screenOrientation="portrait"
                android:theme="@style/AppTheme.NoActionBar" />

        <!-- cache directory file provider paths -->
        <provider
                android:name="androidx.core.content.FileProvider"
                android:authorities="${applicationId}.provider"
                android:exported="false"
                android:grantUriPermissions="true">
            <meta-data
                    android:name="android.support.FILE_PROVIDER_PATHS"
                    android:resource="@xml/file_paths" />
        </provider>
    </application>

</manifest>
5.	Add Dexter and uCrop dependencies to your app/build.gradle
dependencies {
    //...
		implementation 'com.github.yalantis:ucrop:2.2.3'
implementation "com.karumi:dexter:5.0.0"


}
6.	Copy ImagePickerActivity.kt to your project.
7.	Launch ImagePickerActivity by passing required intent data. Once the image is cropped, you can received the path of the cropped image in onActivityResult method.
8.	class MainActivity : AppCompatActivity() {
    private val TAG = MainActivity::class.java.simpleName
    val REQUEST_IMAGE = 100
    private lateinit var imgProfile: ImageView
    private lateinit var img_plus: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        imgProfile = findViewById(R.id.img_profile)
        img_plus = findViewById(R.id.img_plus)
        loadProfileDefault()
        // Clearing older images from cache directory
        // don't call this line if you want to choose multiple images in the same activity
        // call this once the bitmap(s) usage is over
        ImagePickerActivity.clearCache(this)
        imgProfile.setOnClickListener {
            onProfileImageClick()
        }
        img_plus.setOnClickListener {
            onProfileImageClick()
        }
    }

    private fun loadProfileDefault() {

        Glide.with(this).load(R.drawable.baseline_account_circle_black_48)
            .into(imgProfile!!)
        imgProfile!!.setColorFilter(ContextCompat.getColor(this, R.color.profile_default_tint))
    }

    private fun onProfileImageClick() {
        Dexter.withActivity(this)
            .withPermissions(Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE)
            .withListener(object : MultiplePermissionsListener {
                override fun onPermissionsChecked(report: MultiplePermissionsReport) {
                    if (report.areAllPermissionsGranted()) {
                        showImagePickerOptions()
                    }

                    if (report.isAnyPermissionPermanentlyDenied) {
                        showSettingsDialog()
                    }
                }

                override fun onPermissionRationaleShouldBeShown(
                    permissions: List<PermissionRequest>,
                    token: PermissionToken
                ) {
                    token.continuePermissionRequest()
                }
            }).check()
    }

    /**
     * Showing Alert Dialog with Settings option
     * Navigates user to app settings
     * NOTE: Keep proper title and message depending on your app
     */
    private fun showSettingsDialog() {
        val builder = AlertDialog.Builder(this@MainActivity)
        builder.setTitle(getString(R.string.dialog_permission_title))
        builder.setMessage(getString(R.string.dialog_permission_message))
        builder.setPositiveButton(getString(R.string.go_to_settings)) { dialog, which ->
            dialog.cancel()
            openSettings()
        }
        builder.setNegativeButton(getString(android.R.string.cancel)) { dialog, _s -> dialog.cancel() }
        builder.show()

    }

    // navigating user to app settings
    private fun openSettings() {
        val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
        val uri = Uri.fromParts("package", packageName, null)
        intent.data = uri
        startActivityForResult(intent, 101)

    }

    private fun showImagePickerOptions() {
        ImagePickerActivity.showImagePickerOptions(this, object : ImagePickerActivity.PickerOptionListener {
            override fun onTakeCameraSelected() {
                launchCameraIntent()
            }

            override fun onChooseGallerySelected() {
                launchGalleryIntent()
            }
        })
    }

    private fun launchGalleryIntent() {
        val intent = Intent(this@MainActivity, ImagePickerActivity::class.java)
        intent.putExtra(ImagePickerActivity.INTENT_IMAGE_PICKER_OPTION, ImagePickerActivity.REQUEST_GALLERY_IMAGE)

        // setting aspect ratio
        intent.putExtra(ImagePickerActivity.INTENT_LOCK_ASPECT_RATIO, true)
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_X, 1) // 16x9, 1x1, 3:4, 3:2
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_Y, 1)
        startActivityForResult(intent, REQUEST_IMAGE)

    }

    private fun launchCameraIntent() {
        val intent = Intent(this@MainActivity, ImagePickerActivity::class.java)
        intent.putExtra(ImagePickerActivity.INTENT_IMAGE_PICKER_OPTION, ImagePickerActivity.REQUEST_IMAGE_CAPTURE)

        // setting aspect ratio
        intent.putExtra(ImagePickerActivity.INTENT_LOCK_ASPECT_RATIO, true)
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_X, 1) // 16x9, 1x1, 3:4, 3:2
        intent.putExtra(ImagePickerActivity.INTENT_ASPECT_RATIO_Y, 1)

        // setting maximum bitmap width and height
        intent.putExtra(ImagePickerActivity.INTENT_SET_BITMAP_MAX_WIDTH_HEIGHT, true)
        intent.putExtra(ImagePickerActivity.INTENT_BITMAP_MAX_WIDTH, 1000)
        intent.putExtra(ImagePickerActivity.INTENT_BITMAP_MAX_HEIGHT, 1000)

        startActivityForResult(intent, REQUEST_IMAGE)

    }

    @SuppressLint("MissingSuperCall")
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        if (requestCode == REQUEST_IMAGE) {
            if (resultCode == Activity.RESULT_OK) {
                val uri = data!!.getParcelableExtra<Uri>("path")
                try {
                    // You can update this bitmap to your server
                    val bitmap = MediaStore.Images.Media.getBitmap(this.contentResolver, uri)
                    // loading profile image from local cache
                    loadProfile(uri!!.toString())
                } catch (e: IOException) {
                    e.printStackTrace()
                }

            }
        }
    }

    private fun loadProfile(url: String) {
        Log.d(TAG, "Image cache path: $url")
        Glide.with(this).load(url)
            .into(imgProfile!!)
        imgProfile!!.setColorFilter(ContextCompat.getColor(this, android.R.color.transparent))
    }
}


Once the Uri is received, you can create a bitmap and send to your server or preview on the screen.
.
