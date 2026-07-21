# Mobile App (iOS & Android)

> Mobile App không có luồng Kết nối/Gỡ kết nối trên Dashboard. Việc tích hợp được thực hiện qua WebView trong mã nguồn ứng dụng.

## Platform Overview

| Platform | Method |
|----------|--------|
| Flutter | WebView + `loadHtmlString` — `webview_flutter` package |
| iOS | WKWebView + `loadHTMLString` — Native `WebKit` framework |
| Android | WebView + `loadDataWithBaseURL` — Native `android.webkit.WebView` |

Module: `https://app.aichatbot.com.vn/chatbox_v3.js`

---

## Flutter SDK

### Dependencies

```yaml
dependencies:
  webview_flutter: ^4.0.0
  webview_flutter_android: ^4.0.0
  webview_flutter_wkwebview: ^3.25.1
  file_picker: ^11.0.0          # optional — file uploads
  permission_handler: ^12.0.3   # optional — camera/mic
  app_links: ^7.0.0             # optional — deep links
```

### Full-Screen Chatbot Screen

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';
import 'package:webview_flutter_android/webview_flutter_android.dart';
import 'package:file_picker/file_picker.dart';
import 'package:permission_handler/permission_handler.dart';

class ChatbotScreen extends StatefulWidget {
  const ChatbotScreen({super.key});

  @override
  State<ChatbotScreen> createState() => _ChatbotScreenState();
}

class _ChatbotScreenState extends State<ChatbotScreen> {
  WebViewController? _controller;
  bool _isLoading = true;

  static const _deepLinkSchemes = ['yourapp://', 'https://yourdomain.com'];

  @override
  void initState() {
    super.initState();
    _initWebView();
  }

  Future<void> _initWebView() async {
    await Permission.microphone.request();
    await Permission.camera.request();

    final controller = WebViewController()
      ..setJavaScriptMode(JavaScriptMode.unrestricted)
      ..addJavaScriptChannel(
        'ConsoleLogger',
        onMessageReceived: (message) {
          debugPrint('[WebView] ${message.message}');
        },
      )
      ..setNavigationDelegate(
        NavigationDelegate(
          onNavigationRequest: (request) {
            final url = request.url;
            for (final scheme in _deepLinkSchemes) {
              if (url.startsWith(scheme)) {
                _handleDeepLink(url);
                return NavigationDecision.prevent;
              }
            }
            if (url.startsWith('https://aichatbot.com.vn') ||
                url.startsWith('https://app.aichatbot.com.vn') ||
                url.startsWith('https://cdn.aichatbot.com.vn')) {
              return NavigationDecision.navigate;
            }
            return NavigationDecision.prevent;
          },
        ),
      )
      ..loadHtmlString(_chatbotHtml, baseUrl: 'https://app.aichatbot.com.vn');

    if (controller.platform is AndroidWebViewController) {
      final platform = controller.platform as AndroidWebViewController;
      platform.setOnShowFileSelector(_onFileSelector);
      platform.setOnPlatformPermissionRequest((req) => req.grant());
      AndroidWebViewController.enableDebugging(true);
    }

    _controller = controller;
    setState(() => _isLoading = false);
  }

  Future<List<String>> _onFileSelector(FileSelectorParams params) async {
    FileType fileType = FileType.any;
    List<String>? allowedExtensions;
    if (params.acceptTypes.any((t) => t.startsWith('image/'))) {
      fileType = FileType.custom;
      allowedExtensions = ['jpg', 'jpeg', 'png', 'gif', 'webp'];
    } else if (params.acceptTypes.any((t) => t.startsWith('audio/'))) {
      fileType = FileType.audio;
    } else if (params.acceptTypes.any((t) => t.startsWith('video/'))) {
      fileType = FileType.video;
    }
    try {
      final result = await FilePicker.pickFiles(
        type: fileType,
        allowedExtensions: allowedExtensions,
        allowMultiple: params.mode == FileSelectorMode.openMultiple,
        withData: true,
      );
      if (result != null && result.files.isNotEmpty) {
        return result.files.map((f) {
          if (f.bytes != null && f.name.isNotEmpty) {
            final tmp = File('${Directory.systemTemp.path}/${f.name}');
            tmp.writeAsBytesSync(f.bytes!);
            return tmp.uri.toString();
          }
          return f.path!;
        }).toList();
      }
    } catch (e) {
      debugPrint('File picker error: $e');
    }
    return [];
  }

  void _handleDeepLink(String url) {
    final uri = Uri.parse(url);
    Navigator.pop(context);
    Future.delayed(const Duration(milliseconds: 300), () {
      if (mounted) {
        debugPrint('Deep link: $uri');
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Chat')),
      body: _isLoading || _controller == null
          ? const Center(child: CircularProgressIndicator())
          : WebViewWidget(controller: _controller!),
    );
  }
}

const _chatbotHtml = '''
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #ffffff; }
    ai-fullchatbot { display: block; width: 100%; height: 100%; }
  </style>
</head>
<body>
<script type="module">
import Chatbot from 'https://app.aichatbot.com.vn/chatbox_v3.js';
Chatbot.initFull({
    chatbot: 'chatbot_name',
    apiHost: 'https://app.aichatbot.com.vn',
    config: {},
    onRequest: async (request) => {
        // const accessToken = getAccessToken();
        // const headers = new Headers(request.headers);
        // headers.append('Authorization', `Bearer ${accessToken}`);
        // request.headers = headers;
    },
    observersConfig: {
        observeUserInput: (userInput) => {},
        observeMessages: (messages) => {},
        observeLoading: (loading) => {},
    },
    theme: {
        chatWindow: {
            showTitle: true,
            showAgentMessages: true,
            title: 'My Brand',
            titleAvatarSrc: 'https://cdn.aichatbot.com.vn/chatbot_name/ai-agent.png',
            backgroundImage: 'https://cdn.aichatbot.com.vn/chatbot_name/bg.jpg',
            welcomeMessage: '👋 Hello!',
            errorMessage: '',
            backgroundColor: '#ffffff',
            fontSize: 16,
            starterPrompts: [],
            starterPromptFontSize: 15,
            clearChatOnReload: true,
            sourceDocsTitle: 'Source:',
            renderHTML: true,
            botMessage: {
                backgroundColor: '#f7f8ff',
                textColor: '#303235',
                showAvatar: true,
                avatarSrc: 'https://cdn.aichatbot.com.vn/chatbot_name/ai-agent.png',
            },
            userMessage: {
                backgroundColor: '#3B81F6',
                textColor: '#ffffff',
                showAvatar: true,
                avatarSrc: 'https://app.aichatbot.com.vn/usericon.png',
            },
            textInput: {
                placeholder: 'Your question?',
                backgroundColor: '#ffffff',
                textColor: '#303235',
                sendButtonColor: '#3B81F6',
                maxChars: 256,
                maxCharsWarningMessage: 'Maximum 256 characters',
                autoFocus: true,
                sendMessageSound: true,
                sendSoundLocation: 'https://app.aichatbot.com.vn/send_message.mp3',
                receiveMessageSound: true,
                receiveSoundLocation: 'https://app.aichatbot.com.vn/receive_message.mp3',
            },
            feedback: { color: '#303235' },
            dateTimeToggle: { date: false, time: false },
            followUpQuestions: true,
            footer: {
                textColor: '#303235',
                text: 'Powered by',
                company: 'yourcompany.com',
                companyLink: 'yourcompany.com',
            },
            disclaimer: {
                title: 'Disclaimer',
                message: 'By using this chatbot, you agree to our Terms & Conditions',
            },
        },
    },
});
</script>
<ai-fullchatbot></ai-fullchatbot>
</body>
</html>
''';
```

### Floating Button (Home Screen)

```dart
MaterialApp(
  builder: (context, child) {
    return Stack(
      children: [
        child ?? const SizedBox.expand(),
        Positioned(
          right: 20, bottom: 20,
          child: GestureDetector(
            onTap: () => Navigator.push(
              context,
              MaterialPageRoute(builder: (_) => const ChatbotScreen()),
            ),
            child: Container(
              width: 56, height: 56,
              decoration: BoxDecoration(
                color: const Color(0xFFf67c01),
                shape: BoxShape.circle,
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withValues(alpha: 0.3),
                    blurRadius: 12, offset: const Offset(0, 4),
                  ),
                ],
              ),
              child: const Icon(Icons.auto_awesome, color: Colors.white, size: 28),
            ),
          ),
        ),
      ],
    );
  },
);
```

---

## iOS SDK

### ViewController

```swift
import UIKit
import WebKit
import AVFoundation

class ChatbotViewController: UIViewController, WKNavigationDelegate, WKUIDelegate {

    private var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()
        setupWebView()
        loadChatbot()
    }

    private func setupWebView() {
        let config = WKWebViewConfiguration()
        config.allowsInlineMediaPlayback = true
        config.mediaTypesRequiringUserActionForPlayback = []

        webView = WKWebView(frame: view.bounds, configuration: config)
        webView.navigationDelegate = self
        webView.uiDelegate = self
        webView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        view.addSubview(webView)
    }

    private func loadChatbot() {
        AVCaptureDevice.requestAccess(for: .audio) { _ in }
        AVCaptureDevice.requestAccess(for: .video) { _ in }

        webView.loadHTMLString(chatbotHTML, baseURL: URL(string: "https://app.aichatbot.com.vn"))
    }

    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction,
                  decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
        if let url = navigationAction.request.url {
            if url.absoluteString.hasPrefix("yourapp://") ||
               url.absoluteString.hasPrefix("https://yourdomain.com") {
                handleDeepLink(url)
                decisionHandler(.cancel)
                return
            }
        }
        decisionHandler(.allow)
    }

    func webView(_ webView: WKWebView, decidePolicyFor navigationResponse: WKNavigationResponse,
                  decisionHandler: @escaping (WKNavigationResponsePolicy) -> Void) {
        decisionHandler(.allow)
    }

    func webView(_ webView: WKWebView, runOpenPanelWith parameters: WKOpenPanelParameters,
                  initiatedByFrame frame: WKFrameInfo,
                  completionHandler: @escaping ([URL]?) -> Void) {
        var picker: UIImagePickerController?

        if parameters.allowsMultipleSelection {
            if #available(iOS 14, *) {
                var config = PHPickerConfiguration()
                config.selectionLimit = 0
                config.filter = .any(of: [.images, .videos])
                let phpicker = PHPickerViewController(configuration: config)
                phpicker.delegate = self
                self.filePickerCompletion = completionHandler
                present(phpicker, animated: true)
                return
            }
        }

        picker = UIImagePickerController()
        picker?.sourceType = .photoLibrary
        picker?.delegate = self
        self.filePickerCompletion = completionHandler
        present(picker!, animated: true)
    }

    private var filePickerCompletion: (([URL]?) -> Void)?

    private func handleDeepLink(_ url: URL) {
        print("Deep link: \(url.absoluteString)")
    }
}

private let chatbotHTML = """
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #ffffff; }
    ai-fullchatbot { display: block; width: 100%; height: 100%; }
  </style>
</head>
<body>
<script type="module">
import Chatbot from 'https://app.aichatbot.com.vn/chatbox_v3.js';
Chatbot.initFull({
    chatbot: 'chatbot_name',
    apiHost: 'https://app.aichatbot.com.vn',
    theme: { /* same config as init() */ },
});
</script>
<ai-fullchatbot></ai-fullchatbot>
</body>
</html>
"""
```

### Info.plist

```xml
<key>NSPhotoLibraryUsageDescription</key>
<string>Access needed to upload photos in chat</string>
<key>NSCameraUsageDescription</key>
<string>Access needed to take photos for chat</string>
<key>NSMicrophoneUsageDescription</key>
<string>Access needed for voice recording</string>
```

### App Transport Security (ATS)

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

Remove or restrict this in production.

---

## Android SDK

### Activity

```kotlin
package com.your.app

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import android.webkit.*
import android.widget.FrameLayout
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat

class ChatbotActivity : AppCompatActivity() {

    private lateinit var webView: WebView

    private val requestMultiplePermissions =
        registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { _ ->
            loadChatbot()
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        webView = WebView(this).apply {
            settings.javaScriptEnabled = true
            settings.domStorageEnabled = true
            settings.mediaPlaybackRequiresUserGesture = false
            settings.allowFileAccess = true
            settings.allowContentAccess = true
            settings.mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW

            webViewClient = object : WebViewClient() {
                override fun shouldOverrideUrlLoading(
                    view: WebView?, request: WebResourceRequest?
                ): Boolean {
                    request?.url?.let { url ->
                        val urlStr = url.toString()
                        if (urlStr.startsWith("yourapp://") ||
                            urlStr.startsWith("https://yourdomain.com")) {
                            handleDeepLink(url)
                            return true
                        }
                    }
                    return false
                }
            }

            webChromeClient = object : WebChromeClient() {
                override fun onShowFileChooser(
                    webView: WebView?,
                    filePathCallback: ValueCallback<Array<Uri>>?,
                    fileChooserParams: FileChooserParams?
                ): Boolean {
                    return handleFileChooser(filePathCallback, fileChooserParams)
                }
            }
        }

        setContentView(FrameLayout(this).apply {
            addView(webView, FrameLayout.LayoutParams(
                FrameLayout.LayoutParams.MATCH_PARENT,
                FrameLayout.LayoutParams.MATCH_PARENT
            ))
        })

        if (hasPermissions()) loadChatbot() else requestPermissions()
    }

    private fun hasPermissions(): Boolean {
        return ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) ==
                PackageManager.PERMISSION_GRANTED &&
               ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) ==
                PackageManager.PERMISSION_GRANTED
    }

    private fun requestPermissions() {
        requestMultiplePermissions.launch(
            arrayOf(Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO)
        )
    }

    private fun loadChatbot() {
        webView.loadDataWithBaseURL(
            "https://app.aichatbot.com.vn",
            CHATBOT_HTML, "text/html", "UTF-8", null
        )
    }

    private var uploadMessage: ValueCallback<Array<Uri>>? = null

    private fun handleFileChooser(
        callback: ValueCallback<Array<Uri>>?,
        params: FileChooserParams?
    ): Boolean {
        uploadMessage?.onReceiveValue(null)
        uploadMessage = callback

        val intent = params?.createIntent() ?: Intent(Intent.ACTION_GET_CONTENT).apply {
            type = "*/*"
            putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true)
        }

        fileChooserLauncher.launch(intent)
        return true
    }

    private val fileChooserLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
            if (result.resultCode == RESULT_OK) {
                uploadMessage?.onReceiveValue(
                    FileChooserParams.parseResult(result.resultCode, result.data)
                )
            } else {
                uploadMessage?.onReceiveValue(null)
            }
            uploadMessage = null
        }

    private fun handleDeepLink(uri: Uri) {
        // Parse uri.path and navigate to the appropriate screen
    }

    companion object {
        private const val CHATBOT_HTML = """
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body { width: 100%; height: 100%; overflow: hidden; background: #ffffff; }
    ai-fullchatbot { display: block; width: 100%; height: 100%; }
  </style>
</head>
<body>
<script type="module">
import Chatbot from 'https://app.aichatbot.com.vn/chatbox_v3.js';
Chatbot.initFull({
    chatbot: 'chatbot_name',
    apiHost: 'https://app.aichatbot.com.vn',
    theme: { /* same config as init() */ },
});
</script>
<ai-fullchatbot></ai-fullchatbot>
</body>
</html>
"""
    }
}
```

### WebView Config Notes

| Setting | Value | Reason |
|---------|-------|--------|
| `javaScriptEnabled` | `true` | Required for SDK |
| `domStorageEnabled` | `true` | Required for SDK |
| `mediaPlaybackRequiresUserGesture` | `false` | Auto-play audio/video |
| `allowFileAccess` | `true` | File upload support |
| `mixedContentMode` | `MIXED_CONTENT_ALWAYS_ALLOW` | Load HTTPS SDK from plain `baseUrl` |

### Min API Level

- Android **5.0 (API 21)** minimum
- `compileSdk` 33+ for `READ_MEDIA_IMAGES` / `READ_MEDIA_AUDIO` permissions

---

## Deep Linking

The chatbot can send links in messages that open specific screens in your app.

### Link Format

```
Custom scheme:   yourapp://path
Universal link:  https://yourdomain.com/path
```

### WebView Interception (Mobile)

```dart
// Flutter
onNavigationRequest: (request) {
  if (request.url.startsWith('yourapp://') ||
      request.url.startsWith('https://yourdomain.com')) {
    _handleDeepLink(request.url);
    return NavigationDecision.prevent;
  }
}
```

```swift
// iOS
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction,
              decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    if url.absoluteString.hasPrefix("yourapp://") {
        handleDeepLink(url)
        decisionHandler(.cancel)
        return
    }
    decisionHandler(.allow)
}
```

```kotlin
// Android
override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {
    if (urlStr.startsWith("yourapp://")) {
        handleDeepLink(url)
        return true
    }
    return false
}
```

### Route Handling

```dart
static void handleNavigation(BuildContext context, Uri uri) {
  final path = uri.path.isEmpty || uri.path == '/'
      ? '/${uri.host}' : uri.path;

  switch (path) {
    case '/home':
      Navigator.pushNamedAndRemoveUntil(context, '/home', (_) => false);
    case '/accounts':
      Navigator.pushNamed(context, '/accounts');
    case '/cards':
      Navigator.pushNamed(context, '/cards');
    case '/chatbot':
      Navigator.pushNamed(context, '/chatbot');
    default:
      Navigator.pushNamedAndRemoveUntil(context, '/home', (_) => false);
  }
}
```

### Universal Links (iOS)

**File:** `https://yourdomain.com/.well-known/apple-app-site-association`
```json
{
  "applinks": {
    "apps": [],
    "details": [{
      "appID": "TEAM_ID.com.your.bundle",
      "paths": ["/accounts", "/cards", "/deposits", "/loan", "/chatbot", "/home/*"]
    }]
  }
}
```

**Entitlement:** `ios/Runner/Runner.entitlements`
```xml
<key>com.apple.developer.associated-domains</key>
<array>
    <string>applinks:yourdomain.com</string>
</array>
```

### App Links (Android)

**File:** `https://yourdomain.com/.well-known/assetlinks.json`
```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.your.package",
    "sha256_cert_fingerprints": ["SHA256_FINGERPRINT"]
  }
}]
```

**Intent Filter:**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="yourapp" android:host="*"/>
</intent-filter>
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:scheme="https" android:host="yourdomain.com"/>
</intent-filter>
```

---

## Permissions

| Permission | Platform | Purpose |
|------------|----------|---------|
| `INTERNET` | Android | Load SDK data |
| `CAMERA` | Both | Take photos for upload |
| `RECORD_AUDIO` (Android) / `MICROPHONE` (iOS) | Both | Voice recording |
| `READ_MEDIA_IMAGES` / `READ_MEDIA_AUDIO` | Android 13+ | Upload media |
| `READ_EXTERNAL_STORAGE` | Android ≤ 12 | Upload files |
| `NSPhotoLibraryUsageDescription` | iOS | Upload images |

Android:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO"/>
```

iOS (Info.plist):
```xml
<key>NSCameraUsageDescription</key>
<string>...</string>
<key>NSMicrophoneUsageDescription</key>
<string>...</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>...</string>
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Chatbot not rendering | Invalid `chatbot` ID | Verify with aichatbot.com.vn support |
| | `baseUrl` not set | Always set `baseUrl` to `https://app.aichatbot.com.vn` |
| | Content Security Policy blocks SDK | Add `script-src https://app.aichatbot.com.vn` to CSP |
| WebView blank page | JavaScript disabled | Set `JavaScriptMode.unrestricted` / `javaScriptEnabled = true` |
| File upload fails (Android) | `onShowFileChooser` not overridden | Implement `WebChromeClient.onShowFileChooser` |
| Deep link not intercepted | Navigation delegate missing | Add `shouldOverrideUrlLoading` handler in WebView client |
| iOS universal link not working | AASA not served at `/.well-known/` | Deploy valid JSON, verify with Apple's validation tool |
| | Wrong TEAM_ID or bundle ID | Match `appID` format: `TEAM_ID.com.your.bundle` |
| Android app link not verified | SHA256 fingerprint mismatch | Run `keytool -list -v -keystore ...`, update `assetlinks.json` |
| Audio/video not playing | `mediaPlaybackRequiresUserGesture = true` | Set to `false` |
| iOS HTTP load blocked | ATS blocks non-HTTPS | Add `NSAllowsArbitraryLoads` (dev only) |
| SDK module fails to load | CORS not configured | Ensure server serves correct CORS headers for SDK CDN |

### Testing Your Integration

**Mobile:** Use platform tools:
- **Android:** `chrome://inspect` — debug WebView remotely
- **iOS:** Safari → Develop → [Device] → [WebView] — debug WKWebView
