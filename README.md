## Android期中实验
116052019045 马瑞奇

## 1.实验目标

完成基于SQLite的记事本，具有增加记事，查看记事，修改记事，删除记事，查询记事等基本功能

## 2.实验结果

一、基本功能：

1.在每条记事添加时间戳

2.加入搜索框，可进行搜索

3.点击记事项目即可编辑，点击垃圾桶图标即可删除

二、附加功能：

a.新增起始页面，并精化ui，提高用户使用体验

b.新增笔记分类功能，用5个颜色来将笔记分为最多5种类别


app起始界面

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:orientation="vertical"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:background="@color/white">

              <ImageView
                  android:scaleType="fitXY"
                android:layout_width="fill_parent"
                android:layout_height="fill_parent"
                android:src="@drawable/ic_welcome"/>

</LinearLayout>
<!--已完成-->


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN
                , WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_welcome);
        handler.sendEmptyMessageDelayed(0,5000);
    }

    int time = 1;
    private Handler handler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            toMainInterface();
            super.handleMessage(msg);
        }
    };

    public void toMainInterface() {
        Intent intent=new Intent(AppWelcomeActivity.this,MainActivity.class);
        startActivity(intent);
        finish();
    }


    public void onResume() {
        super.onResume();
    }
    public void onPause() {
        super.onPause();
    }
}

显示笔记本的列表（主界面）


@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
Toolbar toolbar =  findViewById(R.id.toolbar);
setSupportActionBar(toolbar);
myHandler=new MyHandler(this);
FloatingActionButton fab =  findViewById(R.id.fab);
fab.setOnClickListener(new View.OnClickListener() {
@Override
public void onClick(View view) {
Intent intent=new Intent(MainActivity.this,CreateNewNoteActivity.class);
startActivity(intent);
}
});
recyclerView=findViewById(R.id.myRecyclerView);
Context context=MainActivity.this;
dbHelper=new DBHelper(context);
db=dbHelper.getReadableDatabase();
showNotes();
myAdapter=new MyAdapter(this,noteList);
recyclerView.setAdapter(myAdapter);
recyclerView.setLayoutManager(new StaggeredGridLayoutManager(2,StaggeredGridLayoutManager.VERTICAL));
recyclerView.setItemAnimator(new DefaultItemAnimator());
recyclerView.setHasFixedSize(true);
myAdapter.setOnItemCLickListener(new OnItemCLickListener() {
@Override
public void onItemClick(View view, int position) {
int id=noteList.get(position).getId();
Bundle b=new Bundle();
b.putInt("id",id);
b.putInt("position",position);
Intent intent=new Intent(MainActivity.this,EditNoteActivity.class);
intent.putExtras(b);
startActivity(intent);
}
});
}

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        SearchManager searchManager=(SearchManager)getSystemService(Context.SEARCH_SERVICE);
        SearchView searchView=(SearchView)menu.findItem(R.id.ab_search).getActionView();
        searchView.setSearchableInfo(searchManager.getSearchableInfo(getComponentName()));
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement


        return super.onOptionsItemSelected(item);
    }
    public void showNotes(){
        Cursor cursor=db.query("notes",null,null,null,null,null,null,null);
        if(cursor.moveToFirst()){
            do {
                int nid=cursor.getInt(0);
                String ntitle=cursor.getString(1);
                String nbody=cursor.getString(2);
                String date=cursor.getString(3);
                String tagcolor=cursor.getString(4);
                Note note=new Note(nid,ntitle,nbody,date,tagcolor);
                noteList.add(note);
            }while (cursor.moveToNext());
        }
    }
    public void updateView(){
        int count=myAdapter.getItemCount();
        count++;
        String scount=Integer.toString(count);
        Cursor cursor=db.rawQuery("select *from notes order by id desc limit 0,1",null);
        if(cursor.moveToFirst()){
                int nid=cursor.getInt(0);
                String ntitle=cursor.getString(1);
                String nbody=cursor.getString(2);
                String date=cursor.getString(3);
                String tagcolor=cursor.getString(4);
                Note note=new Note(nid,ntitle,nbody,date,tagcolor);
                noteList.add(note);
        }
        myAdapter.notifyDataSetChanged();

    }
    private static class MyHandler extends Handler {
        WeakReference<MainActivity> contentWeakReference;
        public MyHandler(MainActivity activity){
            contentWeakReference=new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            MainActivity activity=contentWeakReference.get();
            switch (msg.what){
                case 0x123:
                    if(activity!=null){
                        activity.updateView();
                    }
                    break;
                case 0x124:
                    activity.noteList.set(msg.arg1,(Note)msg.obj);
                    activity.myAdapter.notifyDataSetChanged();
                    break;
                case 0x125:
                    activity.noteList.remove(msg.arg1);
                    activity.myAdapter.notifyDataSetChanged();
                    break;
            }
        }
    }
}

搜索笔记（模糊搜索）

public class SearchResultActivity extends AppCompatActivity {
private List<Note> noteList=new ArrayList<>();
private MyAdapter myAdapter;
private DBHelper dbHelper;
private SQLiteDatabase db;
private RecyclerView recyclerView;
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.search_result);
String SearchContent=getIntent().getStringExtra(SearchManager.QUERY);
recyclerView=findViewById(R.id.myRecyclerView);
Context context=SearchResultActivity.this;
dbHelper=new DBHelper(context);
db=dbHelper.getReadableDatabase();
showNotes(SearchContent);
myAdapter=new MyAdapter(this,noteList);
recyclerView.setAdapter(myAdapter);
recyclerView.setLayoutManager(new LinearLayoutManager(this));
recyclerView.setItemAnimator(new DefaultItemAnimator());
recyclerView.setHasFixedSize(true);
myAdapter.setOnItemCLickListener(new OnItemCLickListener() {
@Override
public void onItemClick(View view, int position) {
int id=noteList.get(position).getId();
Bundle b=new Bundle();
b.putInt("id",id);
b.putInt("position",position);
Intent intent=new Intent(SearchResultActivity.this,EditNoteActivity.class);
intent.putExtras(b);
startActivity(intent);
finish();
}
});
}
public void showNotes(String str){
Cursor cursor=db.rawQuery("select * from notes where title like '%"+str+"%'",null);
if(cursor.moveToFirst()){
do {
int nid=cursor.getInt(0);
String ntitle=cursor.getString(1);
String nbody=cursor.getString(2);
String date=cursor.getString(3);
String tag=cursor.getString(4);
Note note=new Note(nid,ntitle,nbody,date,tag);
noteList.add(note);
}while (cursor.moveToNext());
}
}
}

创建标题与笔记本内容

public class CreateNewNoteActivity extends AppCompatActivity {
private SQLiteDatabase db;
private DBHelper dbHelper;
private EditText titleEdit;
private EditText bodyEdit;

    private Button cancelButton;
    private String colortag;
    private ImageView tagimageview;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.createnotelayout);
        titleEdit=findViewById(R.id.editTitle);
        bodyEdit=findViewById(R.id.editBody);
        cancelButton=findViewById(R.id.editCancel);
        tagimageview=findViewById(R.id.tagimage);
        dbHelper=new DBHelper(this);
        db=dbHelper.getWritableDatabase();
        bindListening();
    }
    public void bindListening(){

        cancelButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater=getMenuInflater();
        inflater.inflate(R.menu.menu_create,menu);
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if(id==R.id.create_save){
            String title=titleEdit.getText().toString();
            String body=bodyEdit.getText().toString();
            SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy年MM月dd日 HH:mm");
            Date date=new Date(System.currentTimeMillis());
            String cdate=simpleDateFormat.format(date);
            db.execSQL("insert into notes(title,body,date,tagcolor) values('"+title+"','"+body+"','"+cdate+"','"+colortag+"')");
            Toast.makeText(CreateNewNoteActivity.this,"添加成功!",Toast.LENGTH_SHORT).show();
            MainActivity.myHandler.sendEmptyMessage(0x123);
            finish();
        }

        return super.onOptionsItemSelected(item);
    }

    public void onCheckTag(View v){
        ImageView img = (ImageView) v;
        colortag = img.getTag() + "";
    }
}

修改现有笔记

public class EditNoteActivity extends AppCompatActivity {
private SQLiteDatabase db;
private DBHelper dbHelper;
private EditText titleEdit;
private EditText bodyEdit;

    private Button cancelButton;
    private String tagcolor;
    private int id;
    private int position;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.edit_activity);
        Toolbar toolbar =  findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        Bundle b=getIntent().getExtras();
         id=b.getInt("id");
         position=b.getInt("position");
        titleEdit=findViewById(R.id.editTitle);
        bodyEdit=findViewById(R.id.editBody);
        cancelButton=findViewById(R.id.editCancel);
        dbHelper=new DBHelper(this);
        getDetail(id);
        bindListening();
    }
    public void bindListening(){

        cancelButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
    }
    public void getDetail(int id){
        db=dbHelper.getReadableDatabase();
        Cursor cursor=db.rawQuery("select * from notes where id="+id,null);
        if(cursor.moveToFirst()){
            titleEdit.setText(cursor.getString(1));
            bodyEdit.setText(cursor.getString(2));
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater=getMenuInflater();
        inflater.inflate(R.menu.menu_edit,menu);
        return super.onCreateOptionsMenu(menu);
    }
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if(id==R.id.ed_delete){
            db=dbHelper.getWritableDatabase();
            db.execSQL("delete from notes where id="+this.id);
            Message message=new Message();
            message.arg1=position;
            message.what=0x125;
            MainActivity.myHandler.sendMessage(message);
            finish();
        }

        if(id==R.id.ed_save){
            db=dbHelper.getWritableDatabase();
            String title=titleEdit.getText().toString();
            String body=bodyEdit.getText().toString();
            SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy年MM月dd日 HH:mm");
            Date date=new Date(System.currentTimeMillis());
            String cdate=simpleDateFormat.format(date);
            db.execSQL("update notes set title=?,body=?,date=?,where id="+id,new String[]{title,body,cdate});
            Toast.makeText(EditNoteActivity.this,"修改成功!",Toast.LENGTH_SHORT).show();
            Note note=new Note(id,title,body,cdate,tagcolor);
            Message message=new Message();
            message.what=0x124;
            message.obj=note;
            message.arg1=position;
            MainActivity.myHandler.sendMessage(message);
            finish();
        }

        return super.onOptionsItemSelected(item);
    }
}

附加功能的实现

a.UI美化

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:orientation="vertical"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:background="@color/white">

              <ImageView
                  android:scaleType="fitXY"
                android:layout_width="fill_parent"
                android:layout_height="fill_parent"
                android:src="@drawable/ic_welcome"/>

</LinearLayout>
<!--已完成-->

<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@mipmap/logo"
    tools:context=".MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/content_main" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        app:srcCompat="@drawable/add" />
    <!--<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
        android:paddingBottom="@dimen/cardview_compat_inset_shadow" tools:context=".MainActivity"
        android:background="@mipmap/logo" android:layout_height="match_parent"/>-->
</android.support.design.widget.CoordinatorLayout>

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:orientation="vertical"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:background="@mipmap/fest">
<android.support.design.widget.AppBarLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

    </android.support.design.widget.AppBarLayout>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:hint="标题"
        android:id="@+id/editTitle"
        android:background="@drawable/shape"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:hint="内容"
        android:layout_marginTop="20dp"
        android:id="@+id/editBody"
        android:background="@drawable/shape"/>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        >
        <Button

            android:layout_width="300dp"
            android:layout_height="wrap_content"
            android:text="取消"
            android:layout_gravity="center"
            android:textColor="#ffffff"
            android:id="@+id/editCancel"
            android:background="@drawable/buttonshape"/>

    </LinearLayout>
</LinearLayout>

b.用不同颜色进行Note的分类

@Override
public boolean onOptionsItemSelected(MenuItem item) {
int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if(id==R.id.create_save){
            String title=titleEdit.getText().toString();
            String body=bodyEdit.getText().toString();
            SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy年MM月dd日 HH:mm");
            Date date=new Date(System.currentTimeMillis());
            String cdate=simpleDateFormat.format(date);
            db.execSQL("insert into notes(title,body,date,tagcolor) values('"+title+"','"+body+"','"+cdate+"','"+colortag+"')");
            Toast.makeText(CreateNewNoteActivity.this,"添加成功!",Toast.LENGTH_SHORT).show();
            MainActivity.myHandler.sendEmptyMessage(0x123);
            finish();
        }

        return super.onOptionsItemSelected(item);
    }

    public void onCheckTag(View v){
        ImageView img = (ImageView) v;
        colortag = img.getTag() + "";
    }
}

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#3F51B5</color>
    <color name="colorPrimaryDark">#303F9F</color>
    <color name="colorAccent">#FF4081</color>
    <color name="white">#ffffff</color>
</resources>
