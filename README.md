# MobileDev-BTVN-02
# SINH VIÊN THỰC HIỆN: LƯƠNG HOÀNG VIỆT - K225480106073
# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG TRÊN THIẾT BỊ DI ĐỘNG
# BÀI TẬP 02
# BÀI LÀM
# I. LÝ THUYẾT
## 1. AndroidManifest.xml
Mô tả: Là tệp cấu hình quan trọng nhất của ứng dụng, chứa thông tin cần thiết về ứng dụng cho hệ điều hành Android (tên package, các thành phần Activity, Service, Receiver, Provider).

Khai báo quyền (Permissions): Khai báo bằng thẻ <uses-permission> trước thẻ <application>.

Ví dụ: <uses-permission android:name="android.permission.CAMERA" />.

Mục đích: Thông báo cho hệ thống và người dùng biết ứng dụng cần truy cập tài nguyên nhạy cảm (Camera, vị trí, danh bạ...).

## 2. Vòng đời (Lifecycle) và onCreate

Vòng đời: Là chuỗi trạng thái mà một Activity trải qua (Created, Started, Resumed, Paused, Stopped, Destroyed).

Hàm onCreate(): Là điểm bắt đầu, được gọi khi Activity lần đầu được tạo.

Tại sao cần: Tại đây, ta thực hiện các công việc khởi tạo quan trọng như thiết lập giao diện (setContentView), khởi tạo biến, kết nối cơ sở dữ liệu. Nếu không có, ứng dụng không thể hiển thị giao diện hay chuẩn bị tài nguyên.

## 3. Kiểm tra quyền (Runtime Permissions - Java)

Code:

Java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, REQUEST_CODE);
}
Ý nghĩa: Từ Android 6.0 trở lên, ứng dụng phải xin quyền ngay lúc chạy (runtime) thay vì chỉ khai báo trong Manifest, giúp bảo mật dữ liệu người dùng.

## 4. Giao diện (Layout) và Tài nguyên (Resources)

Tham chiếu tài nguyên: Thay vì hardcode (ví dụ: android:text="Hello"), ta lưu vào tệp strings.xml dưới dạng: <string name="hello_msg">Hello</string>.

Cú pháp tham chiếu: @string/hello_msg.

Ưu điểm: Dễ quản lý, thay đổi nội dung không cần sửa code, dễ dàng hỗ trợ đa ngôn ngữ (Localization).

Hỗ trợ Auto: Hệ thống tự động chọn tài nguyên phù hợp dựa trên thư mục (vd: values-vn cho tiếng Việt, values-night cho Dark Mode). Điều này giúp app tự thích ứng với thiết bị mà không cần code logic phức tạp.

Đối tượng chứa (ViewGroup): Sử dụng LinearLayout.

Các thuộc tính android:orientation="vertical/horizontal" để sắp xếp con theo chiều dọc/ngang.

android:gravity định vị vị trí các thành phần bên trong.

## 5. Tương tác code và Giao diện
Tránh hardcode: Sử dụng getString(R.string.hello_msg) trong Java để lấy chuỗi từ tệp resource. Hệ thống tự động chọn đúng ngôn ngữ/theme đã cấu hình.

Xử lý sự kiện (Click Button):

Layout cần: Gán id cho button (android:id="@+id/myBtn").

Cách 1 (Anonymous Inner Class):

Java
Button btn = findViewById(R.id.myBtn);
btn.setOnClickListener(new View.OnClickListener() {
    @Override public void onClick(View v) { /* Code xử lý */ }
});

Cách 2 (Implements): Class Activity implements View.OnClickListener, sau đó định nghĩa onClick() và gọi btn.setOnClickListener(this).

## 6. Thư mục Assets
Cú pháp truy cập: Sử dụng AssetManager:

Java

InputStream is = getAssets().open("file_name.txt");

Lợi ích: Lưu trữ tệp tĩnh (hình ảnh lớn, phông chữ, file cấu hình, db offline) giúp ứng dụng hoạt động bình thường ngay cả khi không có Internet.

Ứng dụng: Thích hợp cho các app hướng dẫn, từ điển offline, hoặc game có sẵn dữ liệu nội dung cần tải nhanh.

# II. APP 1: SỔ TAY CẤU HÌNH (DỮ LIỆU ASSETS OFFLINE)
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

## 6. Test ứng dụng trên điện thoại

<img width="1260" height="2800" alt="1781277059183_182607293721674533_7657304349673072617_f1d5488f5ed1ff316bdf420d6c800b4f" src="https://github.com/user-attachments/assets/11b159ce-46be-48aa-9e8a-8d4ac68648a6" />

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
        android:text="PHƯƠNG TRÌNH BẬC 2"
        android:textSize="22sp"
        android:textStyle="bold"
        android:gravity="center"
        android:textColor="#1A202C"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="24dp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="@android:color/white"
        android:padding="16dp"
        android:elevation="4dp"
        android:layout_marginBottom="16dp">

        <EditText
            android:id="@+id/edtA"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:hint="Nhập hệ số a"
            android:inputType="numberSigned|numberDecimal"
            android:layout_marginBottom="12dp"/>

        <EditText
            android:id="@+id/edtB"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:hint="Nhập hệ số b"
            android:inputType="numberSigned|numberDecimal"
            android:layout_marginBottom="12dp"/>

        <EditText
            android:id="@+id/edtC"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:hint="Nhập hệ số c"
            android:inputType="numberSigned|numberDecimal"/>
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="@android:color/white"
        android:padding="16dp"
        android:elevation="4dp"
        android:layout_marginBottom="20dp">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="KẾT QUẢ HIỂN THỊ:"
            android:textStyle="bold"
            android:textColor="#2D3748"
            android:textSize="14sp"
            android:layout_marginBottom="6dp"/>
        <TextView
            android:id="@+id/tvResult"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Chưa có dữ liệu tính toán."
            android:textColor="#4A5568"
            android:textSize="15sp"
            android:lineSpacingMultiplier="1.2"/>
    </LinearLayout>

    <Button
        android:id="@+id/btnSolve"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:text="GIẢI VÀ ĐẨY LÊN API"
        android:textSize="16sp"
        android:textStyle="bold"
        android:backgroundTint="#DD6B20"
        android:layout_marginBottom="12dp"/>

    <Button
        android:id="@+id/btnBackMath"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:text="QUAY LẠI TRANG CHÍNH"
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

        // Nút quay lại màn hình chính
        btnBackMath.setOnClickListener(v -> finish());

        // Nút giải phương trình và đẩy API
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
                        ketLuan = (c == 0) ? "Vo so nghiem" : "Vo nghiem";
                        detailRoots = (c == 0) ? "Phuong trinh dung voi moi x" : "Phuong trinh vo nghiem";
                    } else {
                        ketLuan = "1 nghiem";
                        nghiem = -c / b;
                        detailRoots = "x = " + nghiem;
                    }
                } else {
                    double delta = (b * b) - (4 * a * c);
                    if (delta < 0) {
                        ketLuan = "Vo nghiem";
                        detailRoots = "Delta < 0, phuong trinh vo nghiem thuc";
                    } else if (delta == 0) {
                        ketLuan = "Nghiem kep";
                        nghiem = -b / (2 * a);
                        detailRoots = "x1 = x2 = " + nghiem;
                    } else {
                        ketLuan = "2 nghiem phan biet";
                        double x1 = (-b + Math.sqrt(delta)) / (2 * a);
                        double x2 = (-b - Math.sqrt(delta)) / (2 * a);
                        nghiem = x1; 
                        detailRoots = "x1 = " + x1 + "\nx2 = " + x2;
                    }
                }

                // In kết quả toán học ra màn hình (Không dùng ký tự đặc biệt)
                tvResult.setText("Ket luan: " + ketLuan + "\nChi tiet: \n" + detailRoots);
                
                // Kích hoạt luồng đẩy dữ liệu lên Server
                postToApi(a, b, c, ketLuan, nghiem);

            } catch (Exception e) {
                tvResult.setText("Loi: Vui long nhap day du he so hop le!");
            }
        });
    }

    private void postToApi(double a, double b, double c, String ketLuan, double nghiem) {
        Executors.newSingleThreadExecutor().execute(() -> {
            try {
                // Đã thêm dấu "/" ở cuối để chống lỗi 301 Moved Permanently
                URL url = new URL("https://k58kmt.tdh.io.vn/api/");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setRequestMethod("POST");
                
                // Cấu hình Header
                conn.setRequestProperty("Content-Type", "application/json; charset=utf-8");
                conn.setRequestProperty("Accept", "application/json"); 
                conn.setDoOutput(true);

                // Build chuỗi JSON chuẩn với Tên và MSSV
                JSONObject root = new JSONObject();
                root.put("app_by", STUDENT_CODE);

                JSONObject input = new JSONObject();
                input.put("a", a); 
                input.put("b", b); 
                input.put("c", c);
                input.put("name", "Luong Hoang Viet");
                input.put("mssv", "K225480106073");
                root.put("input", input);

                JSONObject output = new JSONObject();
                output.put("ketluan", ketLuan);
                output.put("abc", "xyz");
                output.put("nghiem", nghiem);
                root.put("output", output);

                // Gửi dữ liệu đi
                OutputStream os = conn.getOutputStream();
                byte[] inputBytes = root.toString().getBytes(StandardCharsets.UTF_8);
                os.write(inputBytes, 0, inputBytes.length);
                os.flush(); 
                os.close();

                // Lấy mã Code trả về từ Server
                int responseCode = conn.getResponseCode();
                
                // Đọc nội dung phản hồi của Server
                BufferedReader br;
                if (100 <= responseCode && responseCode <= 399) {
                    br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                } else {
                    br = new BufferedReader(new InputStreamReader(conn.getErrorStream()));
                }
                
                StringBuilder response = new StringBuilder();
                String line;
                while ((line = br.readLine()) != null) response.append(line);
                br.close();

                // In kết quả API lên màn hình hiển thị
                runOnUiThread(() -> {
                    String currentText = tvResult.getText().toString();
                    if (responseCode == 200 || responseCode == 201) {
                        tvResult.setText(currentText + "\n\n[API THANH CONG - Code " + responseCode + "]\nServer phan hoi: " + response.toString());
                    } else {
                        tvResult.setText(currentText + "\n\n[API BAO LOI - Code " + responseCode + "]\nChi tiet: " + response.toString());
                    }
                });
                
                conn.disconnect();
                
            } catch (Exception e) {
                // In ra màn hình nếu sập mạng hoặc lỗi kết nối
                runOnUiThread(() -> {
                    String currentText = tvResult.getText().toString();
                    tvResult.setText(currentText + "\n\n[LOI MANG / HE THONG]\nChi tiet: " + e.getMessage());
                });
                Log.e("API_ERROR", "Loi he thong: ", e);
            }
        });
    }
}
```

## 6. Layout Màn hình 3 (WebView)

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
## 7. Code Java Màn hình 3

app/src/main/java/com.example.app2/WebActivity.java

```
package com.example.app2;

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.widget.Button;
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

        if (getSupportActionBar() != null) {
            getSupportActionBar().hide();
        }

        // ĐOẠN NÀY LÀ KHỞI TẠO NÚT QUAY LẠI ÔNG BỊ SÓT NHÉ
        Button btnBackWeb = findViewById(R.id.btnBackWeb);
        btnBackWeb.setOnClickListener(v -> finish());

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

## 8. Test trên điện thoại

### a) Screen 1

<img width="1260" height="2800" alt="1781277470377_182607293721674533_7657304349673072617_23443f776a4cde355a843b2d39bcd2e0" src="https://github.com/user-attachments/assets/d5123b05-22e4-4082-87bd-6f1560495b7b" />

### b) Screen 2

Test xem tính toán đã ổn định chưa:

<img width="1260" height="2800" alt="1781277470461_182607293721674533_7657304349673072617_11cd77d9af31ef5f971f7a9164c69630" src="https://github.com/user-attachments/assets/246f7e08-f2f9-48e4-898f-ce7a4837d7e5" />

Truy cập https://k58kmt.tdh.io.vn/ xem đã gửi nội dung tính toán lên chưa:
<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/2be5e3e7-4ec0-4d2f-bc11-4d06c2ef4953" />

### c) Screen 3

Test xem đã truy cập được https://k58kmt.tdh.io.vn/?masv= hay chưa:

<img width="1260" height="2800" alt="1781277470547_182607293721674533_7657304349673072617_3d3a9ca112c7618872f5bcc7f5370a76" src="https://github.com/user-attachments/assets/23ff3e30-354d-46fb-831a-719b317bcd4f" />

Truy cập https://k58kmt.tdh.io.vn/ xem đã tương tác được hay chưa: 
<img width="1920" height="1140" alt="image" src="https://github.com/user-attachments/assets/5a2ff84e-37a0-4f51-aad8-6c342cfe2389" />










