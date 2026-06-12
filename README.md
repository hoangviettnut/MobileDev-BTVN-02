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








