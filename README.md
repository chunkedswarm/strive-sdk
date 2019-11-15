# Strive SDK

## What is the Strive SDK?

The Strive SDK are native bindings for Android and iOS to enable WebRTC-based live streaming CDN offloading.

The SDK itself is written in Go and cross compiled to ARM.
The API is available in Java for Android and Swift/ObjC for iOS.

## Which players are supported ?

The Strive SDK is compatible with all players as long as HLS or MPEG-DASH is used for the transport level.

To achieve this the SDK acts as a software proxy and intercepts player requests to offload approx. 70%-80% of the required traffic on average to other vieweres.

## What do I need to get started ?

Checkout [releases](https://github.com/chunkedswarm/strive-sdk/releases) and select a matching package to download.

For Android there is a single **AAR** file and for iOS there is a single **Framework** folder.

## How to integrate with Android ?

Lets start with a small Android app using the ExoPlayer (you can use any player you like).

1. Please add the **Android AAR** to your Android Studio project as a dependency
2. Use the Strive SDK according to the snippet inside your own project
    1. Define Strive-related variable and credentials
    2. Initialize the SDK once on startup
    3. Rewrite your normal HLS/MPEG-DASH CDN urls to go through the SDK proxy
    4. Store SDK information (session, etc.) on app pause (to detect recurring viewers)
    5. Done!

```Java
package io.strivetech.androidsdk;

import android.app.Activity;
import android.net.Uri;
import android.os.Bundle;

import com.devbrackets.android.exomedia.listener.OnPreparedListener;
import com.devbrackets.android.exomedia.ui.widget.VideoView;

import p2pdnsdk.P2pdnsdk;
import p2pdnsdk.SDK;


public class MainActivity extends Activity implements OnPreparedListener {

    // 1. Declare SDK variables
    private final String userAgent = "strive-sdk-android/2.0.0 " + System.getProperty("http.agent"); // The user agent (important for reporting)
    private final String originBaseURL = "XXX"; // The CDN base url (e.g. https://yourcdn.com)
    private final String accountID = "XXX"; // Your Strive AccountID which can be found inside the dashboard
    private final String swarmID = "XXX"; // Your Strive SwarmID which can be found inside the dashboard
    private SDK sdk; // Global reference to Strive SDK

    private VideoView videoView;

    private void initSDK() {
        try {
            // This is important to make sure that recurring viewers are not counted twice
            // On app close, we store the **strive-source-token** and here we read it again to resume the client session
            String sourceToken = getPreferences(Context.MODE_PRIVATE).getString("strive-source-token", "");
            
            // Debug (optional)
            System.out.println("Using user agent: " + userAgent);
            if (sourceToken.equals("")) {
                System.out.println("Creating new source token");
            } else {
                System.out.println("Resuming source token: " + sourceToken);
            }

            // Initialize the Strive SDK with the predefined variables
            sdk = P2pdnsdk.resumeFromAPI(P2pdnsdk.ModePublicCloud,
                    userAgent,
                    originBaseURL,
                    sourceToken,
                    accountID,
                    swarmID);
        } catch (Exception e) {
            // If the init fails, no worries, the client will simply consume the stream via the CDN only
            System.err.println("Failed to create strive sdk: " + e);
        }
    }

    private void storeSDKInfo() {
        // The SDK was not initialized for some reason, there is nothing to store yet
        if (sdk == null) {
            return;
        }

        // Get the current **strive-source-token** from the sdk
        String sourceToken = sdk.sourceToken();

        System.out.println("Storing source token: " + sourceToken);

        // Store it for later app usages
        getPreferences(Context.MODE_PRIVATE).edit().putString("strive-source-token", sourceToken).apply();
    } 

    private String rewriteURL(String url) {
        // The SDK was not initialized for some reason, just use the real CDN url
        if (sdk == null) {
            return url;
        }

        // Rewrite the CDN url to match the SDK proxy
        return sdk.rewriteURL(url);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 2. Init the SDK once on startup
        initSDK();

        // 3. Rewrite your HLS urls using the sdk and use them in your player -> Done!
        String url = rewriteURL("XXX.m3u8");

        videoView = findViewById(R.id.video_view);
        videoView.setOnPreparedListener(this);
        videoView.setVideoURI(Uri.parse(url));
    }

    @Override
    public void onPrepared() {
        videoView.start();
    }
    
    @Override
    protected void onPause() {
        super.onPause();

        // 4. Finally, store the SDK info on pause
        storeSDKInfo();
    }
}
```
