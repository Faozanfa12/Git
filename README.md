//AdapterHotel
class AdapterHotel(private val data: List<ModelHotel>) :
    RecyclerView.Adapter<AdapterHotel.HotelViewHolder>() {
    class HotelViewHolder(private val binding: ItemHotelBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(hotel: ModelHotel) {
            binding.tvNama.text = hotel.namaHotel
            binding.tvLokasi.text = hotel.lokasiHotel
            binding.tvHarga.text = hotel.hargaHotel
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HotelViewHolder {
        val binding = ItemHotelBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return HotelViewHolder(binding)
    }

    override fun onBindViewHolder(holder: HotelViewHolder, position: Int) {
        val data = data[position]
        holder.bind(data)
    }

    override fun getItemCount(): Int = data.size
}


//HotelActivity
class HotelActivity : AppCompatActivity() {
    private lateinit var binding: ActivityHotelBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityHotelBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.rvHotel.layoutManager = LinearLayoutManager(this)
        binding.rvHotel.setHasFixedSize(true)
        showData()
        tambahData()
    }

    private fun showData() {
        val dataRef = FirebaseDatabase.getInstance().getReference("Hotel")
        dataRef.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                val hotelList = mutableListOf<ModelHotel>()
                val adapter = AdapterHotel(hotelList)
                for (dataSnapshot in snapshot.children) {
                    val hotelKey = dataSnapshot.getValue(ModelHotel::class.java)
                    hotelKey?.let {
                        hotelList.add(it)
                        Log.d("cek data firebase", it.toString())
                    }
                }
                if (hotelList.isNotEmpty()) {
                    binding.rvHotel.adapter = adapter
                } else {
                    Toast.makeText(this@HotelActivity, "Data not found",
                        Toast.LENGTH_LONG).show()
                }
            }
            override fun onCancelled(snapshot: DatabaseError) {
// Handle onCancelled
            }
        })
    }
    private fun tambahData(){
        val database: FirebaseDatabase = FirebaseDatabase.getInstance()
        val hotelRef: DatabaseReference = database.getReference("Hotel")
        binding.fabAddHotel.setOnClickListener {
            val dialogView = LayoutInflater.from(this).inflate(R.layout.upload_dialog, null)
            MaterialAlertDialogBuilder(this)
                .setTitle("Tambah Hotel")
                .setView(dialogView)
                .setPositiveButton("Tambah") { dialog, _ ->
                    val namaHotel =
                        dialogView.findViewById<EditText>(R.id.editTextnamaHotel).text.toString()
                    val hargaHotel =
                        dialogView.findViewById<EditText>(R.id.editTexthargaHotel).text.toString()
                    val lokasiHotel =
                        dialogView.findViewById<EditText>(R.id.editTextlokasiHotel).text.toString()
                    val hotelData = HashMap<String, Any>()
                    hotelData["namaHotel"] = namaHotel
                    hotelData["lokasiHotel"] = lokasiHotel
                    hotelData["lhargaHotel"] = hargaHotel
                    val newHotelRef = hotelRef.push()
                    newHotelRef.setValue(hotelData)
                    dialog.dismiss()
                }
                .setNegativeButton("Batal") { dialog, _ ->
                    dialog.dismiss()
                }
                .show()
        }
    }
}


//MainActivity
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        var binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val database = FirebaseDatabase.getInstance()
        val myRef = database.getReference("restoran")

        binding.btnHotel.setOnClickListener {
            val intent = Intent(this, HotelActivity::class.java)
            startActivity(intent)
        }
    }
}


//ModelHotel
data class ModelHotel(
    val namaHotel: String? = null,
    val lokasiHotel: String? = null,
    val hargaHotel: String? = null,
    val gambarHotel: String? = null
)

//activiy_hotel.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp"
    android:orientation="vertical"
    tools:context=".HotelActivity">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/rvHotel"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/fabAddHotel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end|bottom"
            android:layout_margin="16dp"
            android:backgroundTint="@color/white"
            android:src="@drawable/baseline_add_24"
            />
    </FrameLayout>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:width="200dp"
        android:text="Cek reservasi"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.497"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.15" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:width="200dp"
        android:text="reservasi"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.497"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.33" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:width="200dp"
        android:text="buku tamu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.497"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.426" />
</LinearLayout>



//activity_main
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btnHotel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:width="200dp"
        android:text="hotel"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.497"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.238" />

</androidx.constraintlayout.widget.ConstraintLayout>

//item_hotel
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="80dp"
    android:layout_marginStart="12dp"
    android:layout_marginEnd="12dp"
    android:layout_marginTop="8dp"
    app:cardCornerRadius="20dp"
    android:backgroundTint="@color/white">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical">
        <TextView
            android:id="@+id/tvNama"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Nama Restoran"
            android:textColor="@color/black"
            android:textStyle="bold"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:textSize="18sp"/>
        <TextView
            android:id="@+id/tvLokasi"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Lokasi Restoran"
            android:textColor="@color/black"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:textSize="14sp"/>
        <TextView
            android:id="@+id/tvHarga"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Lokasi Restoran"
            android:textColor="@color/black"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:textSize="14sp"/>
    </LinearLayout>
</androidx.cardview.widget.CardView>



//upload_dialog
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    <EditText
        android:id="@+id/editTextnamaHotel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Nama Hotel" />
    <EditText
        android:id="@+id/editTextlokasiHotel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Lokasi Hotel" />
    <EditText
        android:id="@+id/editTexthargaHotel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Harga Hotel" />
</LinearLayout>
