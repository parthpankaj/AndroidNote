# AndroidNote
package com.example.pankaj.notepadapp;

import android.app.AlertDialog;
import android.content.DialogInterface;
import android.content.Intent;
import android.database.Cursor;
import android.os.Bundle;
import android.support.design.widget.CoordinatorLayout;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.EditText;
import android.widget.Toast;
import java.text.SimpleDateFormat;
import java.util.Calendar;
public class DisplayNote extends AppCompatActivity {
    private NDb mydb;
    EditText name;
    EditText content;
    private CoordinatorLayout coordinatorLayout;
    String dateString;
    Bundle extras;
    int id_To_Update = 0;
    Snackbar snackbar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.viewnotepad);
        name = (EditText) findViewById(R.id.txtname);
        content = (EditText) findViewById(R.id.txtcontent);
        coordinatorLayout = (CoordinatorLayout) findViewById(R.id
                .coordinatorLayout);
        mydb = new NDb(this);
        Bundle extras = getIntent().getExtras();
        if (extras != null) {
            int Value = extras.getInt("id");
            if (Value > 0) {
                snackbar = Snackbar
                        .make(coordinatorLayout, "Note Id : "+String.valueOf(Value), Snackbar.LENGTH_LONG);
                snackbar.show();
                Cursor rs = mydb.getData(Value);
                id_To_Update = Value;
                rs.moveToFirst();
                String nam = rs.getString(rs.getColumnIndex(NDb.name));
                String contents = rs.getString(rs.getColumnIndex(NDb.remark));
                if (!rs.isClosed()) {
                    rs.close();
                }
                name.setText((CharSequence) nam);
                content.setText((CharSequence) contents);
            }
        }
    }
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        Bundle extras = getIntent().getExtras();
        if (extras != null) {
            int Value = extras.getInt("id");
            getMenuInflater().inflate(R.menu.display_menu, menu);
        }
        return true;
    }
    public boolean onOptionsItemSelected(MenuItem item) {
        super.onOptionsItemSelected(item);
        switch (item.getItemId()) {
            case R.id.Delete:
                AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setMessage(R.string.DeleteNote)
                        .setPositiveButton("YES",
                                new DialogInterface.OnClickListener() {
                                    public void onClick(DialogInterface dialog,
                                                        int id) {
                                        mydb.deleteNotes(id_To_Update);
                                        Toast.makeText(DisplayNote.this, "Deleted Successfully",Toast.LENGTH_SHORT).show();
                                        Intent intent = new Intent(
                                                getApplicationContext(),
                                                MyNotes.class);
                                        startActivity(intent);
                                        finish();
                                    }
                                })
                        .setNegativeButton("NO",
                                new DialogInterface.OnClickListener() {
                                    public void onClick(DialogInterface dialog,
                                                        int id) {
                                    }
                                });
                AlertDialog d = builder.create();
                d.setTitle("Are you sure");
                d.show();
                return true;
            case R.id.Save:
                Bundle extras = getIntent().getExtras();
                Calendar c = Calendar.getInstance();
                System.out.println("Current time => " + c.getTime());
                SimpleDateFormat df = new SimpleDateFormat("dd-MMM-yyyy");
                String formattedDate = df.format(c.getTime());
                dateString = formattedDate;
                if (extras != null) {
                    int Value = extras.getInt("id");
                    if (Value > 0) {
                        if (content.getText().toString().trim().equals("")
                                || name.getText().toString().trim().equals("")) {
                            snackbar = Snackbar
                                    .make(coordinatorLayout, "Please fill in name of the note", Snackbar.LENGTH_LONG);
                            snackbar.show();
                        } else {
                            if (mydb.updateNotes(id_To_Update, name.getText()
                                    .toString(), dateString, content.getText()
                                    .toString())) {
                                snackbar = Snackbar
                                        .make(coordinatorLayout, "Your note Updated Successfully!!!", Snackbar.LENGTH_LONG);
                                snackbar.show();
                            } else {
                                snackbar = Snackbar
                                        .make(coordinatorLayout, "There's an error. That's all I can tell. Sorry!", Snackbar.LENGTH_LONG);
                                snackbar.show();
                            }
                        }
                    } else {
                        if (content.getText().toString().trim().equals("")
                                || name.getText().toString().trim().equals("")) {
                            snackbar = Snackbar
                                    .make(coordinatorLayout, "Please fill in name of the note", Snackbar.LENGTH_LONG);
                            snackbar.show();
                        } else {
                            if (mydb.insertNotes(name.getText().toString(), dateString,
                                    content.getText().toString())) {
                                snackbar = Snackbar
                                        .make(coordinatorLayout, "Added Successfully.", Snackbar.LENGTH_LONG);
                                snackbar.show();
                            } else {
                                snackbar = Snackbar
                                        .make(coordinatorLayout, "Unfortunately Task Failed.", Snackbar.LENGTH_LONG);
                                snackbar.show();
                            }
                        }
                    }
                }
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
    @Override
    public void onBackPressed() {
        Intent intent = new Intent(
                getApplicationContext(),
                MyNotes.class);
        startActivity(intent);
        finish();
        return;
    }
}
-----------
package com.example.pankaj.notepadapp;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.DatabaseUtils;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import java.util.ArrayList;
import java.util.HashMap;
public class NDb extends SQLiteOpenHelper {


    public static final String dbname = "MyNotes.db";
    public static final String _id = "_id";
    public static final String name = "name";
    public static final String remark = "remark";
    public static final String dates = "dates";
    public static final String mynotes = "mynotes";
    private HashMap hp;
    SQLiteDatabase db;

    public NDb(Context context) {
        super(context, dbname, null, 1);
    }

            @Override
    public void onCreate(SQLiteDatabase db) {
        // TODO Auto-generated method stub
        db.execSQL("create table mynotes"
                + "(_id integer primary key, name text,remark text,dates text)");
    }
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // TODO Auto-generated method stub
        db.execSQL("DROP TABLE IF EXISTS " + mynotes);
        onCreate(db);
    }

    public Cursor fetchAll() {
        db = this.getReadableDatabase();
        Cursor mCursor = db.query(mynotes, new String[] { "_id", "name",
                "dates", "remark" }, null, null, null, null, null);
        if (mCursor != null) {
            mCursor.moveToFirst();
        }
        return mCursor;
    }
    public boolean insertNotes(String name, String dates, String remark) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues contentValues = new ContentValues();
        contentValues.put("name", name);
        contentValues.put("dates", dates);
        contentValues.put("remark", remark);
        db.insert(mynotes, null, contentValues);
        return true;
    }
    public Cursor getData(int id) {
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor z = db.rawQuery("select * from " + mynotes + " where _id=" + id
                + "", null);
        return z;
    }
    public int numberOfRows() {
        SQLiteDatabase db = this.getReadableDatabase();
        int numRows = (int) DatabaseUtils.queryNumEntries(db, mynotes);
        return numRows;
    }
    public boolean updateNotes(Integer id, String name, String dates,
                               String remark) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues contentValues = new ContentValues();
        contentValues.put("name", name);
        contentValues.put("dates", dates);
        contentValues.put("remark", remark);
        db.update(mynotes, contentValues, "_id = ? ",
                new String[] { Integer.toString(id) });
        return true;
    }
    public Integer deleteNotes(Integer id) {
        SQLiteDatabase db = this.getWritableDatabase();
        return db.delete(mynotes, "_id = ? ",
                new String[] { Integer.toString(id) });
    }
    public ArrayList getAll() {
        ArrayList array_list = new ArrayList();
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor res = db.rawQuery("select * from " + mynotes, null);
        res.moveToFirst();
        while (res.isAfterLast() == false) {
            array_list.add(res.getString(res.getColumnIndex("_id")));
            array_list.add(res.getString(res.getColumnIndex(remark)));
            array_list.add(res.getString(res.getColumnIndex(dates)));
            array_list.add(res.getString(res.getColumnIndex(name)));
            res.moveToNext();
        }
        return array_list;
    }
    ------------
    package com.example.pankaj.notepadapp;

        import android.app.AlertDialog;
        import android.content.Context;
        import android.content.Intent;
        import android.database.Cursor;
        import android.os.Bundle;
        import android.support.design.widget.CoordinatorLayout;
        import android.support.design.widget.FloatingActionButton;
        import android.support.design.widget.Snackbar;
        import android.support.v7.app.AppCompatActivity;
        import android.view.Menu;
        import android.view.MenuItem;
        import android.view.View;
        import android.widget.AdapterView;
        import android.widget.AdapterView.OnItemClickListener;
        import android.widget.LinearLayout;
        import android.widget.ListView;
        import android.widget.SimpleCursorAdapter;
        import android.widget.TextView;
public class MyNotes extends AppCompatActivity {
    private ListView obj;
    NDb mydb;
    FloatingActionButton btnadd;
    ListView mylist;
    Menu menu;
    AlertDialog.Builder alertDialogBuilder;
    AlertDialog alertDialog;
    Context context = this;
    CoordinatorLayout coordinatorLayout;
    SimpleCursorAdapter adapter;
    Snackbar snackbar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.notedisplay);
        coordinatorLayout = (CoordinatorLayout) findViewById(R.id.coordinatorLayout);
        mydb = new NDb(this);
        btnadd = (FloatingActionButton) findViewById(R.id.btnadd);
        btnadd.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Bundle dataBundle = new Bundle();
                dataBundle.putInt("id", 0);
                Intent intent = new Intent(getApplicationContext(),
                        DisplayNote.class);
                intent.putExtras(dataBundle);
                startActivity(intent);
                finish();
            }
        });
        Cursor c = mydb.fetchAll();
        String[] fieldNames = new String[] { NDb.name, NDb._id, NDb.dates, NDb.remark };
        int[] display = new int[] { R.id.txtnamerow, R.id.txtidrow,
                R.id.txtdate,R.id.txtremark };
        adapter = new SimpleCursorAdapter(this, R.layout.listtemple, c, fieldNames,
                display, 0);
        snackbar = Snackbar
                .make(coordinatorLayout, "You are awesome I think!", Snackbar.LENGTH_LONG);
        snackbar.show();
        mylist = (ListView) findViewById(R.id.listView1);
        mylist.setAdapter(adapter);
        mylist.setOnItemClickListener(new OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> arg0, View arg1, int arg2,
                                    long arg3) {
                LinearLayout linearLayoutParent = (LinearLayout) arg1;
                LinearLayout linearLayoutChild = (LinearLayout) linearLayoutParent
                        .getChildAt(0);
                TextView m = (TextView) linearLayoutChild.getChildAt(1);
                Bundle dataBundle = new Bundle();
                dataBundle.putInt("id",
                        Integer.parseInt(m.getText().toString()));
                Intent intent = new Intent(getApplicationContext(),
                        DisplayNote.class);
                intent.putExtras(dataBundle);
                startActivity(intent);
                finish();
            }
        });
    }
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        this.menu = menu;
        return true;
    }
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        super.onOptionsItemSelected(item);
        switch (item.getItemId()) {
            case R.id.add:
                Bundle dataBundle = new Bundle();
                dataBundle.putInt("id", 0);
                Intent intent = new Intent(getApplicationContext(),
                        DisplayNote.class);
                intent.putExtras(dataBundle);
                startActivity(intent);
                finish();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
}
-------------
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    </android.support.design.widget.AppBarLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#5eaeff"
        android:paddingTop="5dp"
        >

        <ListView
            android:id="@+id/listView1"
            android:dividerHeight="2dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentTop="true" >
        </ListView>

    </RelativeLayout>
    <android.support.design.widget.FloatingActionButton
        android:id="@+id/btnadd"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end|bottom"
        android:layout_margin="16dp"
        android:src="@drawable/plus" />
</android.support.design.widget.CoordinatorLayout>
--------------------
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="50dp"
    android:background="#ffffff"
    android:orientation="vertical"
    android:weightSum="2" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal"
        android:weightSum="2" >

        <TextView
            android:id="@+id/txtnamerow"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginLeft="5dp"
            android:layout_weight="1.2"
            android:text="name"
            android:textColor="#000000"
            android:textSize="20sp" />

        <TextView
            android:id="@+id/txtidrow"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="0"
            android:visibility="gone" />

        <TextView
            android:id="@+id/txtdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginRight="5dp"
            android:layout_weight="0.8"
            android:gravity="right"
            android:text="2/2/2015" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="vertical"
        android:weightSum="2" >

        <TextView
            android:id="@+id/txtremark"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_marginLeft="15dp"
            android:layout_marginRight="15dp"
            android:layout_marginTop="5dp"
            android:layout_weight="2"
            android:maxHeight="50dp"
            android:text="fgvgvkjdvmldngvmdngvmrfrfmmgbn"
            android:textSize="15sp"
            android:visibility="gone" />
    </LinearLayout>

</LinearLayout>

-------------------
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/coordinatorLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    </android.support.design.widget.AppBarLayout>


    <LinearLayout
        android:id="@+id/scrollView1"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:weightSum="10"

        >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:orientation="horizontal"
            android:weightSum="2" >

            <TextView
                android:id="@+id/textView1"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="0.5"
                android:gravity="top|center"
                android:text="NAME" />

            <EditText
                android:id="@+id/txtname"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:gravity="top|center"
                android:inputType="textAutoCorrect"
                android:hint="Name of note here"
                android:layout_weight="1.5"
                >
            </EditText>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="9"
            android:orientation="vertical"
            android:weightSum="2" >

            <EditText
                android:id="@+id/txtcontent"
                android:hint="Your text here"
                android:gravity="top|left"
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="2"
                android:inputType="textMultiLine">
            </EditText>
        </LinearLayout>

    </LinearLayout>
</android.support.design.widget.CoordinatorLayout>
--------------------
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <item
        android:id="@+id/Save"
        android:orderInCategory="100"
        android:title="SAVE"
        app:showAsAction="always"/>
    <item
        android:id="@+id/Delete"
        android:orderInCategory="100"
        android:title="delete"/>
</menu>
------------
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <item
        android:id="@+id/add"
        android:orderInCategory="100"
        android:title="Add new"
        app:showAsAction="always|withText"/>
</menu>
----------
<resources>
    <string name="app_name">Simple Notepad</string>
    <string name="hello_world">Hello world!</string>
    <string name="action_settings">Settings</string>
    <string name="DeleteNote">Are you Sure?</string>
</resources>
