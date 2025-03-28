## Commit 1 Reflection

Berikut adalah code pada commit ke-1:

```rust
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

Kode tersebut merupakan implementasi sederhana dari server HTTP dengan memanfaatkan bahasa pemrograman Rust. Program ini membuat suatu server TCP yang mendengarkan koneksi masuk pada alamat 127.0.0.1 dengan port 7878. Ketika ada koneksi yang masuk, program akan menerima stream data dan meneruskannya ke fungsi `handle_connection`.

Fungsi `handle_connection` ini menerima parameter berupa `TcpStream`, yaitu koneksi TCP aktif dengan _client_. `mut` sendiri digunakan sebagai penanda bahwa _stream_ dapat diubah (_mutable_) selama fungsi berjalan. Kemudian, pada baris pertama, fungsi akan menggunakan `BufReader`, salah satu komponen standar dalam Rust yang menyediakan _buffering_ untuk operasi I/O.

Selanjutnya, fungsi akan menggunakan rangkaian method chaining, diawali dengan `.lines()` untuk membaca data baris per baris dari _stream_. Kemudian, method `.map(|result| result.unwrap())` digunakan untuk mengekstrak String dari setiap Result yang dihasilkan. Lalu, `.take_while(|line| !line.is_empty())` untuk mengambil baris-baris hingga menemukan baris kosong yang menandakan akhir dari header HTTP. Terakhir, `.collect()` akan mengumpulkan seluruh baris tersebut ke dalam `vector` yang disimpan dalam variabel `http_request`. Fungsi kemudian akan mencetak seluruh permintaan HTTP yang telah diproses ke _console_ dengan memanfaatkan `println!`.

Setelah itu, saya menjalankan perintah `cargo run`, lalu mengakses URL `http://127.0.0.1:7878/` dan mendapatkan output berikut:

```console
PS E:\Semester 4\Adpro\W06\hello> cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target\debug\hello.exe`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "Connection: keep-alive",
    "Cache-Control: max-age=0",
    "sec-ch-ua: \"Chromium\";v=\"134\", \"Not:A-Brand\";v=\"24\", \"Google Chrome\";v=\"134\"",
    "sec-ch-ua-mobile: ?0",
    "sec-ch-ua-platform: \"Windows\"",
    "Upgrade-Insecure-Requests: 1",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-User: ?1",
    "Sec-Fetch-Dest: document",
    "Accept-Encoding: gzip, deflate, br, zstd",
    "Accept-Language: en-US,en;q=0.9,id-ID;q=0.8,id;q=0.7",
]
```

Output tersebut merupakan hasil permintaan HTTP yang diterima oleh server, dimulai dengan baris `GET / HTTP/1.1` yang menandakan permintaan GET untuk halaman root menggunakan protokol HTTP versi 1.1. Kemudian, ada `Host: 127.0.0.1:7878` yang menentukan tujuan _request_ serta `Connection: keep-alive` dan `Cache-Control: max-age=0` yang mengatur perilaku _connection_ dan _caching_. Informasi terkait browser dapat dilihat pada header `sec-ch-ua`, `sec-ch-ua-mobile`, `sec-ch-ua-platform`, dan `User-Agent` yang mengidentifikasi Chrome 134 pada Windows. Kemudian, terdapat `Upgrade-Insecure-Requests: 1` yang menunjukkan preferensi untuk koneksi yang aman. Lalu, terdapat header `Accept` yang menunjukkan tipe konten yang didukung, diikuti oleh header keamanan seperti `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-User`, dan `Sec-Fetch-Dest` yang mengendalikan _behaviour_ dalam pengambilan konten. Terakhir, `Accept-Encoding` menunjukkan algoritma kompresi yang didukung dan `Accept-Language` menunjukkan preferensi bahasa pengguna.

## Commit 2 Reflection

![Commit 2 Screen Capture](/assets/images/commit2.jpg)

```rust
use std::{
    fs,
    ...
};

...

fn handle_connection(mut stream: TcpStream) {
    ...

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    stream.write_all(response.as_bytes()).unwrap();
}
```

Pada commit 2, fungsi `handle_connection` telah kita perbarui untuk tidak sekedar hanya membaca HTTP _request_, tetapi juga mengirimkan respons HTTP kembali ke klien. Kode baru ini menambahkan fungsionalitas untuk membuat respons HTTP yang lengkap dengan status line `HTTP/1.1 200 OK1`, membaca isi file `hello.html` dengan menggunakan fungsi `fs::read_to_string`, menghitung panjang konten dengan `contens.len()`, dan kemudian merangkai semua komponen tersebut menjadi String respons yang valid menggunakan `format!`. Kemudian, respons HTTP yang telah dirangkai tersebut akan dikirimkan kembali ke klien melalui _stream_ TCP dengan memanfaatkan metode `write_all`, sehingga sekarang server kita tidak hanya menerima _request_ tetapi juga mengirimkan HTML _page_ sebagai respons.

Pada saat menjalankan perintah `cargo run`, saya mendapatkan output berikut:

```console
PS E:\Semester 4\Adpro\W06\hello> cargo run
   Compiling hello v0.1.0 (E:\Semester 4\Adpro\W06\hello)
warning: unused variable: `http_request`
  --> src\main.rs:19:9
   |
19 |     let http_request: Vec<_> = buf_reader
   |         ^^^^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_http_request`
   |
   = note: `#[warn(unused_variables)]` on by default

warning: `hello` (bin "hello") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.49s
     Running `target\debug\hello.exe`
```

Output tersebut menunjukkan bahwa program Rust berhasil di-_compile_, tetapi menghasilkan satu peringatan (_warning_). Peringatan tersebut memberitahu bahwa variabel `http_request` yang didefinisikan dalam fungsi `handle_connection` tidak digunakan dalam kode. _Compiler_ menyarankan kita untuk menambahkan awalan _underscore_ jika memang variabel tersebut sengaja tidak digunakan. Meskipun ada peringatan tersebut, program tetap berhasil di-_compile_ dan dijalankan dalam mode _development_ dengan profil debug yang _unoptimized_.

## Commit 3 Reflection

![Commit 3 Screen Capture](/assets/images/commit3.jpg)

Pada commit 3, terdapat beberapa perubahan pada code fungsi `handle_connection` kita, yaitu:

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

Pada perubahan tersebut, server kini dapat membedakan antara request yang valid dan tidak valid dengan memeriksa baris pertama dari permintaan HTTP. Jika permintaan adalah `GET / HTTP/1.1`, server akan merespons dengan status `200 OK` dan mengirimkan konten dari `hello.html`. Namun, jika permintaan berbeda (seperti mengakses halaman yang tidak ada), maka server akan merespons dengan status `404 NOT FOUND` dan mengirimkan konten dari `404.html`. Hal ini memungkinkan server untuk memberikan respons yang sesuai berdasarkan jenis _request_ yang diterima dan meningkatkan fungsionalitas dari server HTTP yang kita buat dibandingkan dengan sebelumnya yang hanya bisa mengirimkan satu jenis respons untuk seluruh permintaan.

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Selanjutnya, dengan memanfaatkan refactoring, fungsi `handle_connection` menjadi lebih ringkas. Perubahan utama terletak pada penggunaan _tuple pattern matching_ untuk menentapkan nilai dari `status_line` dan `filename` dalam satu operasi. Dengan memanfaatkan pendekatan ini, kita dapat menghilangkan duplikasi kode yang terjadi saat membuat respons untuk kedua kasus, karena sekarang kode kita akan membaca file, menghitung panjang konten, dan mengirim respons dalam sekali jalan. Hal ini menerapkan prinsip DRY yang telah dipelajari pada materi sebelumnya. 

## Commit 4 Reflection

Pada commit 4, terdapat beberapa perubahan code sebagai berikut:

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Perubahan pada commit 4 ini menunjukkan pengembangan lanjutan dari fungsi `handle_connection` dengan beberapa peningkatan penting. Pertama, kita mengubah pernyataan `if-else` menjadi ekspresi `match`, dengan melakukan pattern matching pada slice dari string _request line_. Kedua, selain menangani rute utama dan rute yang tidak valid, code kita sekarang juga menangani rute baru `GET /sleep HTTP/1.1`. Terakhir, klausa `_` digunakan pada ekspresi `match` sebagai `catch-all` untuk menangani semua pola permintaan lainnya, memberikan respons 404 untuk semua rute yang tidak kita kenali.

Rute `/sleep` sendiri dibuat untuk mensimulasikan permasalahan pada aplikasi single-threaded. Dengan menambahkan delay 10 detik menggunakan `thread::sleep`, rute ini memperlihatkan bagaimana satu _request_ yang memakan waktu lama akan memblokir seluruh server. Saat browser pertama mengakses `/sleep`, server akan menjadi tidak responsif terhadap request lain hingga proses delay selesai.

Hal ini terjadi karena dalam arsitektur single-htreaded, server hanya dapat melayani satu koneksi pada satu waktu. Di dalam loop `for stream in listener.incoming()`, setiap koneksi diproses secara _sequential_ dan harus selesai sebelum koneksi berikutnya ditangani. Simulasi ini dengan jelas menunjukkan betapa pentingnya konsep multi-threading untuk layanan web app sejenis.

## Commit 5 Reflection

Pada commit 5, terdapat sedikit perubahan pada `main.rs` dan ada file baru bernama `lib.rs`. Berikut adalah kodenya:

```rust
// main.rs
use hello::ThreadPool;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

```rust
// lib.rs
use std::{
    sync::{mpsc, Arc, Mutex},
    thread::{self, JoinHandle},
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {id} got a job; executing.");
            job();
        });

        Worker { id, thread }
    }
}
```

Berdasarkan kedua kode tersebut, server telah diperbarui menjadi model multithreaded dengan implementasi Thread Pool untuk manangani _concurrency_. Pada `main.rs`, perubahan utama adalah pembuatan instance `ThreadPool` dengan 4 thread worker dan penggunaan metode `pool.execute()` untuk membagi penanganan koneksi ke thread pool. Hal ini memungkinkan server untuk menangani beberapa koneksi secara bersamaan tanpa blocking satu sama lain.

File `lib.rs` sendiri mengimplementasikan infrastruktur thread pool dengan struktur `ThreadPool` yang berisikan kumpulan `Worker` dan sistem komunikasi berbasis channel. Setiap `Worker` memiliki thread yang berjalan dalam _infinite loop_, menunggu pekerjaan dari channel yang dibagikan menggunakan `Arc<Mutex<>>`. Ketika method `execute()` dipanggil, pekerjaan akan dibungkus dalam `Box` dan dikirim melalui channel ke salah satu worker yang tersedia. Implementasi ini memanfaatkan beberapa fitur dalam Rust seperti ownership untuk menciptakan sistem _concurrency_ yang aman dan efisien. Hal ini dibuktikan dengan tidak adanya delay ketika mengakses page lain ketika membuka rute `/sleep`.

## Commit Bonus Reflection

Pada bonus ini, saya melakukan beberapa perubahan, baik pada `main.rs` maupun `lib.rs`. Berikut adalah perubahannya:

```rust
fn main() {
    ...

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        ...
    }
}
```

```rust
pub struct PoolCreationError {
    reason: String,
}

impl fmt::Display for PoolCreationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(
            f,
            "Unable to create ThreadPool: {}", self.reason
        )
    }
}

impl fmt::Debug for PoolCreationError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{{ file: {}, line: {} }}", file!(), line!())
    }
}

impl ThreadPool {
    pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
        if size <= 0 {
            Err(PoolCreationError {
                reason: "Thread count must be a positive nonzero integer".to_string()
            })
        } else {
            let (sender, receiver) = mpsc::channel();
            let receiver = Arc::new(Mutex::new(receiver));

            let mut workers = Vec::with_capacity(size);

            for id in 0..size {
                workers.push(Worker::new(id, Arc::clone(&receiver)));
            }

            Ok(ThreadPool { workers, sender })
        }
    }
    
    ...
}
```

Berdasarkan perubahan tersebut, saya telah mengimplementasikan method `build` pada `ThreadPool` yang menerapkan pola Builder. Method ini menerima parameter ukuran thread pool dan mengembalikan `Result<ThreadPool, PoolCreationError>` yang memungkinkan penanganan error secara eksplisit. Untuk mendukung penanganan error, saya juga mendefinisikan `struct PoolCreationError` dengan implementasi `Display` dan `Debug`, memungkinkan pengguna untuk mendapatkan informasi yang lebih baik tentang mengapa pembuatan thread pool bisa gagal. Dalam implementasi file `main.rs`, saya mengubah cara pembuatan thread pool dari `ThreadPool::new(4)` menjadi `ThreadPool::build(4).unwrap()` untuk mengakomodasi pengembalian `Result`.