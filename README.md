# WORKSHOP
# Develop an android application to create records that store Country names and their respective currencies in the form of a ListView using SQLite DataBase.

# AIM:

Develop an android application to create records that store Country names and their respective currencies in the form of a ListView using SQLite DataBase.

# ALGORITHM:

1.Start the application.

2.Initialize the SQLite database using DatabaseHelper.

3.Create a Country table with ID, Country Name and Currency.

4.Display country records using ListView.

5.Click Add Country to enter new country details.

6.Insert the details into the SQLite database.

7.Select a country from ListView to modify its details.

8.Update the selected record in the database.

9.Delete the selected record when required.

10.Refresh the ListView to display updated records.

11.Stop the application.

# PROGRAM:

# Developed by : Kowshika R
# Register number : 212224220049

## DatabaseHelper.java

```
package com.example.countryapplication;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class Databasehelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "CountryDB";
    private static final int DATABASE_VERSION = 1;

    public static final String TABLE_NAME = "Country";
    public static final String COL_ID = "_id";
    public static final String COL_COUNTRY = "country";
    public static final String COL_CURRENCY = "currency";

    public Databasehelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        String query = "CREATE TABLE " + TABLE_NAME + " (" +
                COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                COL_COUNTRY + " TEXT, " +
                COL_CURRENCY + " TEXT)";

        db.execSQL(query);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db,
                          int oldVersion, int newVersion) {

        db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
        onCreate(db);
    }
}
```

## DBManager.java

```
package com.example.countryapplication;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class DBManager {

    Databasehelper helper;
    SQLiteDatabase db;

    public DBManager(Context context) {
        helper = new Databasehelper(context);
    }

    public void open() {
        db = helper.getWritableDatabase();
    }

    public void close() {
        helper.close();
    }

    // INSERT
    public long insert(String country, String currency) {

        ContentValues values = new ContentValues();

        values.put(Databasehelper.COL_COUNTRY, country);
        values.put(Databasehelper.COL_CURRENCY, currency);

        return db.insert(Databasehelper.TABLE_NAME, null, values);
    }

    // DISPLAY
    public Cursor fetch() {

        return db.query(
                Databasehelper.TABLE_NAME,
                null, null, null, null, null,
                Databasehelper.COL_ID + " DESC"
        );
    }

    // UPDATE
    public int update(int id, String country, String currency) {

        ContentValues values = new ContentValues();

        values.put(Databasehelper.COL_COUNTRY, country);
        values.put(Databasehelper.COL_CURRENCY, currency);

        return db.update(
                Databasehelper.TABLE_NAME,
                values,
                Databasehelper.COL_ID + "=?",
                new String[]{String.valueOf(id)}
        );
    }

    // DELETE
    public int delete(int id) {

        return db.delete(
                Databasehelper.TABLE_NAME,
                Databasehelper.COL_ID + "=?",
                new String[]{String.valueOf(id)}
        );
    }
}
```

## CountryListActivity.java

```
package com.example.countryapplication;

import android.app.Activity;
import android.os.Bundle;
import android.content.Intent;
import android.database.Cursor;
import android.widget.*;
import java.util.ArrayList;

public class CountryListActivity extends Activity {

    ListView listView;
    Button btnAdd;

    DBManager manager;

    ArrayList<String> countries;
    ArrayList<Integer> ids;

    ArrayAdapter<String> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        btnAdd = findViewById(R.id.btnAdd);

        manager = new DBManager(this);

        btnAdd.setOnClickListener(v -> {
            Intent i = new Intent(this, AddCountryActivity.class);
            startActivity(i);
        });

        listView.setOnItemClickListener((parent, view, position, id) -> {

            Intent i = new Intent(this, ModifyCountryActivity.class);

            i.putExtra("id", ids.get(position));

            startActivity(i);
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        displayCountries();
    }

    private void displayCountries() {

        manager.open();

        countries = new ArrayList<>();
        ids = new ArrayList<>();

        Cursor cursor = manager.fetch();

        while (cursor.moveToNext()) {

            int id = cursor.getInt(0);
            String country = cursor.getString(1);
            String currency = cursor.getString(2);

            ids.add(id);

            countries.add(country + " - " + currency);
        }

        cursor.close();
        manager.close();

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                countries
        );

        listView.setAdapter(adapter);
    }
}
```

## AddCountryActivity.java 

```
package com.example.countryapplication;

import android.app.Activity;
import android.os.Bundle;
import android.widget.*;

public class AddCountryActivity extends Activity {

    EditText etCountry, etCurrency;
    Button btnSave;
    DBManager manager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_country);

        etCountry = findViewById(R.id.etCountry);
        etCurrency = findViewById(R.id.etCurrency);
        btnSave = findViewById(R.id.btnSave);

        manager = new DBManager(this);

        btnSave.setOnClickListener(v -> {

            String country = etCountry.getText().toString().trim();
            String currency = etCurrency.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this,
                        "Enter all details",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            manager.open();

            long result = manager.insert(country, currency);

            manager.close();

            if (result != -1) {
                Toast.makeText(this,
                        "Country Added Successfully",
                        Toast.LENGTH_SHORT).show();
                finish();
            } else {
                Toast.makeText(this,
                        "Insertion Failed",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

## ModifyCountryActivity.java

```
package com.example.countryapplication;

import android.app.Activity;
import android.os.Bundle;
import android.database.Cursor;
import android.widget.*;

public class ModifyCountryActivity extends Activity {

    EditText etCountry, etCurrency;
    Button btnUpdate, btnDelete;

    DBManager manager;
    int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_modify_country);

        etCountry = findViewById(R.id.etCountry);
        etCurrency = findViewById(R.id.etCurrency);

        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);

        manager = new DBManager(this);

        countryId = getIntent().getIntExtra("id", -1);

        loadCountry();

        btnUpdate.setOnClickListener(v -> {

            String country = etCountry.getText().toString().trim();
            String currency = etCurrency.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {
                Toast.makeText(this,
                        "Enter all details",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            manager.open();

            int result = manager.update(countryId, country, currency);

            manager.close();

            if (result > 0) {
                Toast.makeText(this,
                        "Updated Successfully",
                        Toast.LENGTH_SHORT).show();
                finish();
            }
        });

        btnDelete.setOnClickListener(v -> {

            manager.open();

            int result = manager.delete(countryId);

            manager.close();

            if (result > 0) {
                Toast.makeText(this,
                        "Deleted Successfully",
                        Toast.LENGTH_SHORT).show();
                finish();
            }
        });
    }

    private void loadCountry() {

        manager.open();

        Cursor cursor = manager.fetch();

        while (cursor.moveToNext()) {

            if (cursor.getInt(0) == countryId) {

                etCountry.setText(cursor.getString(1));
                etCurrency.setText(cursor.getString(2));

                break;
            }
        }

        cursor.close();
        manager.close();
    }
}
```

## activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="20dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="COUNTRY CURRENCY"
        android:textSize="26sp"
        android:textStyle="bold"
        android:gravity="center"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="SQLite Database Management"
        android:textSize="18sp"
        android:layout_marginTop="15dp"/>

</LinearLayout>
```

## MainActivity.java

```
package com.example.countryapplication;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent(
                MainActivity.this,
                CountryListActivity.class
        );

        startActivity(intent);
        finish();
    }
}
```
## AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.CountryApplication">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".CountryListActivity"
            android:exported="false" />

        <activity
            android:name=".AddCountryActivity"
            android:exported="false" />

        <activity
            android:name=".ModifyCountryActivity"
            android:exported="false" />

    </application>

</manifest>
```

# OUTPUT

<img width="1917" height="1147" alt="image" src="https://github.com/user-attachments/assets/73ab1e52-6e91-4074-b4e1-802f5883b240" />

<img width="1917" height="1145" alt="image" src="https://github.com/user-attachments/assets/a6a303ef-71f1-4b47-bc97-fc308b67fef2" />

<img width="1917" height="1136" alt="image" src="https://github.com/user-attachments/assets/d6c62335-5b98-4167-b30c-4f698389923a" />

<img width="1917" height="1147" alt="image" src="https://github.com/user-attachments/assets/996f0519-468a-4e5a-a97d-a2931dc24fd0" />

<img width="1917" height="1143" alt="image" src="https://github.com/user-attachments/assets/32dbc23a-3b9f-41b7-9b43-269404290be0" />

# RESULT

Thus, the Android application was successfully developed using Java and SQLite Database to store, display, insert, update and delete country names and their respective currencies using a ListView.
