# MobileDev-BTVN-02
# SINH VIÊN THỰC HIỆN: LƯƠNG HOÀNG VIỆT - K225480106073
# MÔN HỌC: LẬP TRÌNH TRÊN THIẾT BỊ DI ĐỘNG
# BÀI TẬP 02
# BÀI LÀM
# I. APP 1: SỔ TAY CẤU HÌNH (DỮ LIỆU ASSETS OFFLINE)
## 1. Cấu hình Manifest & Default Activity

app/src/main/AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.app1">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Sổ Tay DevOps"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AppCompat.Light.DarkActionBar">
        
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <!-- Định nghĩa màn hình chính Launcher -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
    </application>
</manifest>
```
## 2. Dữ liệu JSON Máy chủ (Assets)

app/src/main/assets/server_configs.json

```
[
  {
    "name": "Docker Môi Trường",
    "command": "docker-compose up -d --build",
    "desc": "Khởi chạy và xây dựng lại toàn bộ các container dịch vụ chạy ngầm dưới nền."
  },
  {
    "name": "Nginx Reverse Proxy",
    "command": "sudo nginx -t && sudo systemctl restart nginx",
    "desc": "Kiểm tra cú pháp tệp tin ảo và tái khởi động dịch vụ máy chủ web Nginx."
  },
  {
    "name": "Hệ Thống Ubuntu Server",
    "command": "sudo apt update && sudo apt dist-upgrade -y",
    "desc": "Cập nhật danh sách gói phần mềm và nâng cấp bản vá bảo mật cốt lõi cho OS."
  }
]
```

## 3. Giao diện Tùy chỉnh (item_command.xml)

app/src/main/res/layout/item_command.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:background="#1E1E1E"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="#FFFFFF"
        android:textSize="18sp"
        android:textStyle="bold"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:id="@+id/tvCommand"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#000000"
        android:padding="12dp"
        android:textColor="#00E676" 
        android:fontFamily="monospace"
        android:textSize="15sp"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:id="@+id/tvDesc"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="#AAAAAA"
        android:textSize="14sp"/>
</LinearLayout>
```

## 4. Giao diện Màn hình chính

app/src/main/res/layout/activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:fitsSystemWindows="true"
    android:paddingTop="16dp" 
    android:paddingHorizontal="16dp"
    android:background="#121212">

    <TextView
        android:id="@+id/txtAppTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="🛠️ CẨM NANG LỆNH INFRASTRUCTURE"
        android:textColor="#FFFFFF"
        android:textSize="18sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginBottom="24dp"/>

    <ListView
        android:id="@+id/lvServerCommands"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:divider="@android:color/transparent"
        android:dividerHeight="16dp"
        android:scrollbars="none"/>
</LinearLayout>
```
## 5. Xử lý Java

app/src/main/java/com.example.app1/MainActivity.java
```
package com.example.app1;

import android.os.Bundle;
import android.util.Log;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import org.json.JSONArray;
import org.json.JSONObject;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.HashMap;

public class MainActivity extends AppCompatActivity {
    private ListView lvServerCommands;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Ẩn Action Bar
        if (getSupportActionBar() != null) {
            getSupportActionBar().hide();
        }

        lvServerCommands = findViewById(R.id.lvServerCommands);
        executeAssetParsingAlgorithm();
    }

    private void executeAssetParsingAlgorithm() {
        try {
            InputStream inputStream = getAssets().open("server_configs.json");
            int fileSize = inputStream.available();
            byte[] buffer = new byte[fileSize];
            inputStream.read(buffer);
            inputStream.close();

            String rawJsonString = new String(buffer, StandardCharsets.UTF_8);
            JSONArray rootJsonArray = new JSONArray(rawJsonString);

            ArrayList<HashMap<String, String>> dataList = new ArrayList<>();

            for (int i = 0; i < rootJsonArray.length(); i++) {
                JSONObject nodeObject = rootJsonArray.getJSONObject(i);
                HashMap<String, String> map = new HashMap<>();
                map.put("name", "📌 " + nodeObject.getString("name"));
                map.put("command", ">_ " + nodeObject.getString("command"));
                map.put("desc", "📝 " + nodeObject.getString("desc"));
                dataList.add(map);
            }

            String[] fromKeys = {"name", "command", "desc"};
            int[] toViews = {R.id.tvName, R.id.tvCommand, R.id.tvDesc};

            SimpleAdapter adapter = new SimpleAdapter(this, dataList, R.layout.item_command, fromKeys, toViews);
            lvServerCommands.setAdapter(adapter);

        } catch (Exception exception) {
            Log.e("APP1_ERROR", "Lỗi đọc Assets: ", exception);
            Toast.makeText(this, "Lỗi nạp dữ liệu!", Toast.LENGTH_LONG).show();
        }
    }
}
```

## 6. Test Ứng dụng trên máy

<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/09471188-37f5-43b5-b9f0-b3ca267622f8" />

# III. APP 2: KIẾN TRÚC 3 ACTIVITIES & TƯƠNG TÁC HTTP API
## 1. Cấp Quyền Internet & Khai báo Activity

app/src/main/AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.app2"> <!-- Đã đổi đúng tên Package -->

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Hệ Thống Tương Tác API"
        android:theme="@style/Theme.AppCompat.Light.DarkActionBar">
        
        <!-- Màn hình 1 -->
        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- Màn hình 2 & 3 -->
        <activity android:name=".MathActivity" />
        <activity android:name=".WebActivity" />
    </application>
</manifest>
```

## 2. Layout Màn hình 1 (About & Điều hướng)

app/src/main/res/layout/activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="24dp"
    android:fitsSystemWindows="true" 
    android:background="#F0F4F8">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="🎓 THÔNG TIN SINH VIÊN"
        android:textSize="22sp"
        android:textStyle="bold"
        android:textColor="#1A202C"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="32dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="@android:color/white"
        android:padding="24dp"
        android:elevation="4dp" 
        android:layout_marginBottom="40dp">
        
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="👤 Sinh viên: Lương Hoàng Việt" android:textSize="17sp" android:textStyle="bold" android:textColor="#2D3748" android:layout_marginBottom="12dp"/>
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="🆔 Mã SV: K225480106073" android:textSize="16sp" android:textColor="#4A5568" android:layout_marginBottom="12dp"/>
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content" android:text="💻 Chuyên ngành: KTPM" android:textSize="16sp" android:textColor="#4A5568"/>
    </LinearLayout>

    <Button
        android:id="@+id/btnGoMath"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:text="📐 MÀN HÌNH GIẢI TOÁN"
        android:textSize="16sp"
        android:textStyle="bold"
        android:backgroundTint="#3182CE"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/btnGoWeb"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:text="🌐 MÀN HÌNH WEBVIEW"
        android:textSize="16sp"
        android:textStyle="bold"
        android:backgroundTint="#38A169"/>
</LinearLayout>
```

## 3. Code Java Màn hình 1 (Chuyển Activity)

app/src/main/java/com.example.app2/MainActivity.java

```
package com.example.app2;

import android.content.Intent;
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Ẩn thanh Action Bar mặc định
        if (getSupportActionBar() != null) {
            getSupportActionBar().hide();
        }

        Button btnGoMath = findViewById(R.id.btnGoMath);
        Button btnGoWeb = findViewById(R.id.btnGoWeb);

        // Sử dụng Lambda expressions cho gọn
        btnGoMath.setOnClickListener(v -> startActivity(new Intent(this, MathActivity.class)));
        btnGoWeb.setOnClickListener(v -> startActivity(new Intent(this, WebActivity.class)));
    }
}
```
## 4. Layout Màn hình 2 (Giải Toán)

app/src/main/res/layout/activity_math.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:fitsSystemWindows="true"
    android:background="#F0F4F8">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="📐 PHƯƠNG TRÌNH BẬC 2"
        android:textSize="22sp"
        android:textStyle="bold"
        android:gravity="center"
        android:textColor="#1A202C"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="32dp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="@android:color/white"
        android:padding="20dp"
        android:elevation="4dp"
        android:layout_marginBottom="30dp">

        <EditText
            android:id="@+id/edtA"
            android:layout_width="match_parent"
            android:layout_height="55dp"
            android:hint="Nhập hệ số a"
            android:inputType="numberSigned|numberDecimal"
            android:layout_marginBottom="16dp"/>

        <EditText
            android:id="@+id/edtB"
            android:layout_width="match_parent"
            android:layout_height="55dp"
            android:hint="Nhập hệ số b"
            android:inputType="numberSigned|numberDecimal"
            android:layout_marginBottom="16dp"/>

        <EditText
            android:id="@+id/edtC"
            android:layout_width="match_parent"
            android:layout_height="55dp"
            android:hint="Nhập hệ số c"
            android:inputType="numberSigned|numberDecimal"/>
    </LinearLayout>

    <Button
        android:id="@+id/btnSolve"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:text="🚀 GIẢI &amp; ĐẨY LÊN API"
        android:textSize="16sp"
        android:textStyle="bold"
        android:backgroundTint="#DD6B20"
        android:layout_marginBottom="12dp"/>

    <!-- Nút Quay Lại -->
    <Button
        android:id="@+id/btnBackMath"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:text="⬅️ QUAY LẠI TRANG CHÍNH"
        android:textSize="14sp"
        android:textStyle="bold"
        android:textColor="#4A5568"
        android:backgroundTint="#E2E8F0"/>
</LinearLayout>
```
## 5. Code Java Màn hình 2 (Toán & API POST)

app/src/main/java/com.example.app2/MathActivity.java

```
package com.example.app2;

import android.os.Bundle;
import android.util.Log;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import org.json.JSONObject;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.Executors;

public class MathActivity extends AppCompatActivity {
    private final String STUDENT_CODE = "K225480106073";
    private TextView tvResult;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_math);

        if (getSupportActionBar() != null) {
            getSupportActionBar().hide();
        }

        EditText edtA = findViewById(R.id.edtA);
        EditText edtB = findViewById(R.id.edtB);
        EditText edtC = findViewById(R.id.edtC);
        Button btnSolve = findViewById(R.id.btnSolve);
        Button btnBackMath = findViewById(R.id.btnBackMath);
        tvResult = findViewById(R.id.tvResult);

        btnBackMath.setOnClickListener(v -> finish());

        btnSolve.setOnClickListener(v -> {
            try {
                double a = Double.parseDouble(edtA.getText().toString());
                double b = Double.parseDouble(edtB.getText().toString());
                double c = Double.parseDouble(edtC.getText().toString());

                String ketLuan;
                String detailRoots = "";
                double nghiem = 0.0;

                if (a == 0) {
                    if (b == 0) {
                        ketLuan = (c == 0) ? "Vô số nghiệm" : "Vô nghiệm";
                        detailRoots = (c == 0) ? "Phương trình thỏa mãn với mọi x" : "Phương trình không có nghiệm";
                    } else {
                        ketLuan = "1 nghiệm";
                        nghiem = -c / b;
                        detailRoots = "Nghiệm bậc nhất x = " + nghiem;
                    }
                } else {
                    double delta = (b * b) - (4 * a * c);
                    if (delta < 0) {
                        ketLuan = "Vô nghiệm";
                        detailRoots = "Vô nghiệm thực (Δ < 0)";
                    } else if (delta == 0) {
                        ketLuan = "Nghiệm kép";
                        nghiem = -b / (2 * a);
                        detailRoots = "x1 = x2 = " + nghiem;
                    } else {
                        ketLuan = "2 nghiệm PB";
                        double x1 = (-b + Math.sqrt(delta)) / (2 * a);
                        double x2 = (-b - Math.sqrt(delta)) / (2 * a);
                        nghiem = x1; 
                        detailRoots = "x1 = " + x1 + "
x2 = " + x2;
                    }
                }

                // Xuất trực tiếp lên giao diện thay vì gọi Toast
                tvResult.setText("📌 Kết luận: " + ketLuan + "
📋 Chi tiết:
" + detailRoots);
                
                // Vẫn giữ tiến trình truyền tải dữ liệu API ngầm
                postToApi(a, b, c, ketLuan, nghiem);

            } catch (Exception e) {
                tvResult.setText("❌ Lỗi: Vui lòng nhập đầy đủ hệ số hợp lệ!");
            }
        });
    }

    private void postToApi(double a, double b, double c, String ketLuan, double nghiem) {
        Executors.newSingleThreadExecutor().execute(() -> {
            try {
                URL url = new URL("https://k58kmt.tdh.io.vn/api");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setRequestMethod("POST");
                conn.setRequestProperty("Content-Type", "application/json; charset=utf-8");
                conn.setDoOutput(true);

                JSONObject root = new JSONObject();
                root.put("app_by", STUDENT_CODE);

                JSONObject input = new JSONObject();
                input.put("a", a); input.put("b", b); input.put("c", c);
                input.put("name", "hello tắc kè");
                root.put("input", input);

                JSONObject output = new JSONObject();
                output.put("ketluan", ketLuan);
                output.put("abc", "xyz");
                output.put("nghiem", nghiem);
                root.put("output", output);

                OutputStream os = conn.getOutputStream();
                os.write(root.toString().getBytes(StandardCharsets.UTF_8));
                os.close();

                if (conn.getResponseCode() == 200) {
                    BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = br.readLine()) != null) response.append(line);
                    br.close();
                    Log.d("API_SUCCESS", response.toString());
                }
                conn.disconnect();
            } catch (Exception e) {
                Log.e("API_ERROR", "Lỗi mạng: ", e);
            }
        });
    }
}
```

## 6. Layout & Code Java Màn hình 3 (WebView)

app/src/main/res/layout/activity_web.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:fitsSystemWindows="true"
    android:background="#F0F4F8">
    
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="🌐 TRÌNH DUYỆT WEBVIEW"
        android:textSize="18sp"
        android:textStyle="bold"
        android:gravity="center"
        android:textColor="#1A202C"
        android:padding="16dp"
        android:background="@android:color/white"
        android:elevation="4dp"/>

    <WebView
        android:id="@+id/myWebView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
        
    <!-- Nút Quay Lại -->
    <Button
        android:id="@+id/btnBackWeb"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:text="⬅️ QUAY LẠI TRANG CHÍNH"
        android:textSize="14sp"
        android:textStyle="bold"
        android:textColor="#4A5568"
        android:backgroundTint="#E2E8F0"
        android:layout_margin="16dp"/>
</LinearLayout>
```

app/src/main/java/com.example.app2/WebActivity.java

```
package com.example.app2; // Đã đổi chuẩn

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import androidx.appcompat.app.AppCompatActivity;

public class WebActivity extends AppCompatActivity {
    private final String MSV = "K225480106073";

    @SuppressLint("SetJavaScriptEnabled")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_web);

        WebView webView = findViewById(R.id.myWebView);
        
        // Bật JS và ép trang web mở ngay trong lòng app thay vì văng ra Chrome
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        webView.setWebViewClient(new WebViewClient());

        // Định tuyến với mã sinh viên
        String targetUrl = "https://k58kmt.tdh.io.vn?masv=" + MSV;
        webView.loadUrl(targetUrl);
    }
}
```
## 7. Test trên máy ảo








