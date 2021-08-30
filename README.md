# android-kotlin-getLatestLocation
# kotlinのAndoroidネイティブアプリケーションで位置情報を取得する。

## 初めに
　このアプリは、kotlinを使用して位置情報を取得することができます。<br>
Andoroidのアプリで位置情報を取得するのにはGoogleのライブラリ['com.google.android.gms:play-services-location:18.0.0']を使いました。ここでは、このアプリの仕組みとコーディングの流れを説明します。ぜひ読んでください。（誤字脱字があるかもしれませんがご了承ください。）

## アプリの概要
　このアプリの仕組みを説明します。
アプリの動作はこんな感じです。

1. デバイスの位置情報アクセス権限の取得
- AndoroidManifest.xmlにパーミッションを書き込む。
- コードの中でパーミッションが取得できているか確認する。

2. ライブラリを使って位置情報を取得
- 位置情報のアップデート開始（これをすることで、デバイスが位置情報を更新し続けてくれます。）
- 最後に更新された位置情報を取得
- 位置情報のアップデート終了（これをして、アップデートを止める。）

## コーディングの流れ
　ここでは、どういう流れでコーディングするか解説します。
 
1. ライブラリのインストール<br>
　まずは、ライブラリをインストールします。ライブラリはこのコードではcom.google.android.gms:play-services-location:18.0.0が使われていますが、Googleで調べて必ず最新版を使うようにしてください。（app/build.gradle）<br>
最新版でない場合は、Andoroid Studioが黄色くマークしてくれると思います。<br>
　ライブラリのインストールは、まずapp/build.gradleに下のように書き込みます。そうすると、上のほうに青文字で[Sync]が出てきます。それを押すとインストールされると思います。<br>

app/build.gradle（一部抜粋）
```
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation 'androidx.core:core-ktx:1.6.0'
    implementation 'androidx.appcompat:appcompat:1.3.0'
    implementation 'com.google.android.material:material:1.4.0'
    // これを書き込む
    implementation 'com.google.android.gms:play-services-location:18.0.0'
    
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```

　そうしたら、MainActivity.ktでインポートします。
 
 MainAchtivity.kt
 ```
 package com.example.getlatestlocation

import android.Manifest
import android.content.pm.PackageManager
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.core.app.ActivityCompat
// これを書く
import com.google.android.gms.location.*
 ```
 
 そしてビルド、多分ここでエラー吐かなければOK！

2. 位置情報の取得
　まずは、ユーザーから権限を奪取するべくAndoroidManifest.xmlに下のように書き込みます。
 
 AndoroidManifest.xml
 ```
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.INTERNET"/>
 ```
 
 そしたら、MainActivity.ktで権限の確認をします。ここで権限がない場合は、ユーザーに要求してください。
 
 MainActivity.kt
 ```
     private fun fetchLatestLocation() {
        var latestLocation = fusedLocationProviderClient.lastLocation // 最新の位置情報を取得

        // 位置情報のアクセス権限を確認・なければ要求
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
            != PackageManager.PERMISSION_GRANTED &&
            ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION)
            != PackageManager.PERMISSION_GRANTED) {

            // この関数では権限をリクエストしている。
            ActivityCompat.requestPermissions(
                this,
                arrayOf(
                    Manifest.permission.ACCESS_COARSE_LOCATION,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ),
                101
            )
        } else if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(
                    Manifest.permission.ACCESS_COARSE_LOCATION
                ),
                101
            )
        } else if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(
                    Manifest.permission.ACCESS_FINE_LOCATION
                ),
                101
            )
        } else {
            latestLocation.addOnSuccessListener {
                if (it != null) {
                    Toast.makeText(this, "${it.latitude} \n ${it.longitude}", Toast.LENGTH_SHORT).show()
                }
                val locationRequest = createLocationRequest()?:return@addOnSuccessListener
                
                // 位置情報のアップデートを開始
                fusedLocationProviderClient.requestLocationUpdates(
                    locationRequest, // 上で作成されたリクエストを渡す。
                    callback, // 更新された際に行われる処理（上のほうに定義されている。）
                    null
                )
            }
        }
    }
 ```
 そしたらやっとfusedLocationProviderClient.requestLocationUpdates()ができます。（これは、位置情報のアップデートを開始するメソッドです。）
位置取得をしてください。<br>
　ちなみにこのアプリでは、AndoroidのToastという機能を使って下のほうにボヤっと表示するようにしています。

## 終わりに
　位置取得のコードはやることが多く、使われるメソッドも多いですがAndoroidのアプリを作るときは「権限要求　→　権限確認　→　実行」と覚えておけばダイジョブだと思います。詳しい関数の仕様は[公式ドキュメント](https://developer.android.com/training/location/retrieve-current?hl=ja)を確認してください。
