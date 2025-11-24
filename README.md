# 🏥 UTS BASIS DATA — Sistem Informasi Rumah Sakit

Project ini berisi pembuatan sistem basis data rumah sakit yang terdiri dari elemen Rumah Sakit, Dokter, Poliklinik, Jadwal Praktek, Pasien, Resep, dan Kunjungan.
Dokumentasi ini mencakup ERD, daftar tabel, serta seluruh migration & seeder.

## 📌 ERD

📚 Daftar Tabel

Rumah Sakit

Dokter

Poliklinik

Jadwal Praktek

Pasien

Resep

Kunjungan

## 🗂️ Migration & Seeder
### 🏥 Rumah Sakit
Migration: rumah_sakits
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('rumah_sakits', function (Blueprint $table) {
            $table->id();
            $table->string('nama');
            $table->string('alamat');
            $table->string('telepon')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('rumah_sakits');
    }
};
```

Seeder: rumah_sakits
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class RumahSakitSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('rumah_sakits')->insert([
            ['nama' => 'RSUD Sehat Ga Sehat', 'alamat' => 'Jl. Serta Mulia', 'telepon' => '0211234321'],
            ['nama' => 'Klinik Bisa Sehat Lagi', 'alamat' => 'Jl. Kesehatan Melimpah', 'telepon' => '021987666'],
        ]);
    }
}
```

### 🧑‍⚕️ Dokter
Migration: dokters
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('dokters', function (Blueprint $table) {
            $table->id();
            $table->string('nama_dokter');
            $table->string('spesialisasi');
            $table->string('nomor_str')->unique()->nullable();
            $table->string('nomor_telepon')->nullable();
            $table->string('email')->unique()->nullable();

            $table->foreignId('rumah_sakit_id')
                  ->constrained('rumah_sakits')
                  ->onDelete('cascade');

            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('dokters');
    }
};
```

Seeder: DokterSeeder
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use App\Models\RumahSakit;

class DokterSeeder extends Seeder
{
    public function run(): void
    {
        $rumahSakitId = RumahSakit::first()->id ?? null;

        if (!$rumahSakitId) {
            echo "Error: Tidak ada Rumah Sakit ditemukan. Jalankan RumahSakitSeeder terlebih dahulu.\n";
            return;
        }

        DB::table('dokters')->insert([
            [
                'nama_dokter' => 'Dr. Budi Santoso',
                'spesialisasi' => 'Bedah Umum',
                'nomor_str' => 'STR-12345678',
                'nomor_telepon' => '0812111222',
                'email' => 'budi.santoso@rsh.com',
                'rumah_sakit_id' => $rumahSakitId,
                'created_at' => now(),
                'updated_at' => now(),
            ],
        ]);
    }
}
```

### 🏨 Poliklinik
Migration: polikliniks
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('polikliniks', function (Blueprint $table) {
            $table->id();
            $table->string('nama_poli')->unique();
            $table->integer('lantai');
            $table->text('deskripsi')->nullable();

            $table->foreignId('rumah_sakit_id')
                  ->constrained('rumah_sakits')
                  ->onDelete('cascade');

            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('polikliniks');
    }
};
```

Seeder: PoliklinikSeeder
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class PoliklinikSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('polikliniks')->insert([
            ['nama_poli' => 'Poli Anak',    'lantai' => 1, 'deskripsi' => 'Pelayanan kesehatan anak',  'rumah_sakit_id' => 1],
            ['nama_poli' => 'Poli Jantung', 'lantai' => 2, 'deskripsi' => 'Pelayanan jantung',        'rumah_sakit_id' => 1],
            ['nama_poli' => 'Poli Umum',    'lantai' => 1, 'deskripsi' => 'Pelayanan umum',           'rumah_sakit_id' => 2],
        ]);
    }
}
```

### 🕒 Jadwal Praktek
Migration: jadwal_prakteks
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('jadwal_prakteks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('dokter_id')->constrained()->cascadeOnDelete();
            $table->string('hari');
            $table->time('jam_mulai');
            $table->time('jam_selesai');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('jadwal_prakteks');
    }
};
```

Seeder: JadwalPraktekSeeder
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\JadwalPraktek;

class JadwalPraktekSeeder extends Seeder
{
    public function run(): void
    {
        JadwalPraktek::factory()->count(20)->create();
    }
}
```

### 🧍 Pasien
Migration: pasiens
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('pasiens', function (Blueprint $table) {
            $table->id();
            $table->string('nama');
            $table->string('nik')->unique();
            $table->string('alamat');
            $table->date('tanggal_lahir');
            $table->string('telepon')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('pasiens');
    }
};
```

Seeder: PasienSeeder
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Pasien;

class PasienSeeder extends Seeder
{
    public function run(): void
    {
        Pasien::factory()->count(20)->create();
    }
}
```

### 💊 Resep
Migration: reseps
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('reseps', function (Blueprint $table) {
            $table->id();
            $table->foreignId('kunjungan_id')
                  ->constrained('kunjungans')
                  ->cascadeOnDelete();
            $table->text('resep');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('reseps');
    }
};
```

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Resep;

class ResepSeeder extends Seeder
{
    public function run(): void
    {
        Resep::factory()->count(30)->create();
    }
}
```

### 📝 Kunjungan
Migration: kunjungans
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('kunjungans', function (Blueprint $table) {
            $table->id();
            $table->foreignId('pasien_id')->constrained()->cascadeOnDelete();
            $table->foreignId('dokter_id')->constrained()->cascadeOnDelete();
            $table->dateTime('tanggal_kunjungan');
            $table->text('keluhan')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('kunjungans');
    }
};
```

Seeder: KunjunganSeeder
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Kunjungan;

class KunjunganSeeder extends Seeder
{
    public function run(): void
    {
        Kunjungan::factory()->count(30)->create();
    }
}
```
