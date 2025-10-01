import json
import random
import os
import time
from datetime import datetime

class VocabularyTrainer:
    def __init__(self):
        self.vocab_data = {}
        self.load_vocabulary()
        self.stats = {
            'total_attempts': 0,
            'correct_answers': 0,
            'chapters_completed': {},
            'last_session': None
        }
        self.load_stats()
    
    def load_vocabulary(self):
        """Memuat data kosakata dari struktur yang telah diberikan"""
        # Data kosakata akan dimuat di sini
        # Untuk keperluan contoh, saya akan membuat struktur sederhana
        # Dalam implementasi nyata, data lengkap 60 bab akan dimasukkan
        
        # Contoh data untuk demonstrasi
        self.vocab_data = {
            1: {
                'title': '자기소개 (Perkenalan diri)',
                'vocabulary': {
                    1: {'korean': '간호사', 'indonesian': 'Perawat'},
                    2: {'korean': '경찰관', 'indonesian': 'Polisi'},
                    3: {'korean': '근로자', 'indonesian': 'Pekerja'},
                    # ... dan seterusnya untuk semua kosakata bab 1
                }
            },
            2: {
                'title': '생활용품 (Barang kebutuhan sehari-hari)',
                'vocabulary': {
                    1: {'korean': '거울', 'indonesian': 'Cermin'},
                    2: {'korean': '계산기', 'indonesian': 'Kalkulator'},
                    3: {'korean': '가족사진', 'indonesian': 'Foto keluarga'},
                    # ... dan seterusnya untuk semua kosakata bab 2
                }
            },
            # ... dan seterusnya untuk semua 60 bab
        }
    
    def load_stats(self):
        """Memuat statistik pembelajaran"""
        try:
            with open('vocab_stats.json', 'r') as f:
                self.stats = json.load(f)
        except FileNotFoundError:
            self.save_stats()
    
    def save_stats(self):
        """Menyimpan statistik pembelajaran"""
        self.stats['last_session'] = datetime.now().isoformat()
        with open('vocab_stats.json', 'w') as f:
            json.dump(self.stats, f, indent=2)
    
    def display_menu(self):
        """Menampilkan menu utama"""
        print("\n" + "="*50)
        print("PENGHAPAL KOSAKATA BAHASA KOREA")
        print("="*50)
        print("1. Belajar per Bab")
        print("2. Kuis Acak")
        print("3. Review Kosakata")
        print("4. Statistik Belajar")
        print("5. Tes Kemampuan")
        print("6. Keluar")
        print("="*50)
    
    def study_by_chapter(self):
        """Belajar kosakata per bab"""
        print("\nDaftar Bab:")
        for chapter_num, chapter_data in self.vocab_data.items():
            print(f"{chapter_num}. {chapter_data['title']}")
        
        try:
            chapter_choice = int(input("\nPilih bab (1-60): "))
            if chapter_choice not in self.vocab_data:
                print("Bab tidak valid!")
                return
            
            chapter = self.vocab_data[chapter_choice]
            print(f"\nBelajar Bab {chapter_choice}: {chapter['title']}")
            print("-" * 40)
            
            # Tampilkan semua kosakata dalam bab
            for num, vocab in chapter['vocabulary'].items():
                print(f"{num}. {vocab['korean']} - {vocab['indonesian']}")
            
            input("\nTekan Enter untuk melanjutkan...")
            
            # Mode belajar interaktif
            self.interactive_study(chapter_choice)
            
        except ValueError:
            print("Input tidak valid!")
    
    def interactive_study(self, chapter_num):
        """Mode belajar interaktif untuk bab tertentu"""
        chapter = self.vocab_data[chapter_num]
        vocab_list = list(chapter['vocabulary'].items())
        random.shuffle(vocab_list)  # Acak urutan belajar
        
        correct_count = 0
        total_questions = len(vocab_list)
        
        print(f"\nMode Belajar Interaktif - Bab {chapter_num}")
        print("Tekan 'q' untuk kembali ke menu")
        print("-" * 40)
        
        for num, vocab in vocab_list:
            print(f"\nApa arti dari: {vocab['korean']}")
            user_answer = input("Jawaban: ").strip()
            
            if user_answer.lower() == 'q':
                break
            
            if user_answer.lower() == vocab['indonesian'].lower():
                print("✅ Benar!")
                correct_count += 1
            else:
                print(f"❌ Salah! Jawaban yang benar: {vocab['indonesian']}")
            
            self.stats['total_attempts'] += 1
        
        self.stats['correct_answers'] += correct_count
        
        # Update statistik bab
        if chapter_num not in self.stats['chapters_completed']:
            self.stats['chapters_completed'][chapter_num] = 0
        self.stats['chapters_completed'][chapter_num] += 1
        
        print(f"\nHasil: {correct_count}/{total_questions} benar")
        self.save_stats()
    
    def random_quiz(self):
        """Kuis dengan pertanyaan acak dari semua bab"""
        all_vocab = []
        for chapter_data in self.vocab_data.values():
            all_vocab.extend(chapter_data['vocabulary'].items())
        
        if not all_vocab:
            print("Tidak ada kosakata tersedia!")
            return
        
        num_questions = min(20, len(all_vocab))  # Maksimal 20 pertanyaan
        quiz_vocab = random.sample(all_vocab, num_questions)
        
        correct_count = 0
        
        print(f"\nKuis Acak - {num_questions} Pertanyaan")
        print("Tekan 'q' untuk kembali ke menu")
        print("-" * 40)
        
        for num, vocab in quiz_vocab:
            print(f"\nApa arti dari: {vocab['korean']}")
            user_answer = input("Jawaban: ").strip()
            
            if user_answer.lower() == 'q':
                break
            
            if user_answer.lower() == vocab['indonesian'].lower():
                print("✅ Benar!")
                correct_count += 1
            else:
                print(f"❌ Salah! Jawaban yang benar: {vocab['indonesian']}")
            
            self.stats['total_attempts'] += 1
        
        self.stats['correct_answers'] += correct_count
        
        accuracy = (correct_count / num_questions) * 100
        print(f"\nHasil Kuis: {correct_count}/{num_questions} benar ({accuracy:.1f}%)")
        self.save_stats()
    
    def review_vocabulary(self):
        """Review semua kosakata"""
        print("\nMode Review Kosakata")
        print("Pilih opsi review:")
        print("1. Review per Bab")
        print("2. Review Semua Kosakata")
        print("3. Kembali")
        
        choice = input("Pilihan: ").strip()
        
        if choice == '1':
            self.review_by_chapter()
        elif choice == '2':
            self.review_all()
    
    def review_by_chapter(self):
        """Review kosakata per bab"""
        print("\nPilih Bab untuk Direview:")
        for chapter_num, chapter_data in self.vocab_data.items():
            print(f"{chapter_num}. {chapter_data['title']}")
        
        try:
            chapter_choice = int(input("\nPilih bab: "))
            if chapter_choice not in self.vocab_data:
                print("Bab tidak valid!")
                return
            
            chapter = self.vocab_data[chapter_choice]
            print(f"\nReview Bab {chapter_choice}: {chapter['title']}")
            print("=" * 50)
            
            for num, vocab in chapter['vocabulary'].items():
                print(f"{num}. {vocab['korean']} - {vocab['indonesian']}")
            
            input("\nTekan Enter untuk kembali...")
            
        except ValueError:
            print("Input tidak valid!")
    
    def review_all(self):
        """Review semua kosakata dari semua bab"""
        print("\nReview Semua Kosakata")
        print("=" * 60)
        
        for chapter_num, chapter_data in self.vocab_data.items():
            print(f"\nBab {chapter_num}: {chapter_data['title']}")
            print("-" * 40)
            
            for num, vocab in chapter_data['vocabulary'].items():
                print(f"  {vocab['korean']} - {vocab['indonesian']}")
            
            if chapter_num % 5 == 0:  # Jeda setiap 5 bab
                input("\nTekan Enter untuk melanjutkan...")
    
    def show_stats(self):
        """Menampilkan statistik belajar"""
        total_chapters = len(self.vocab_data)
        completed_chapters = len(self.stats['chapters_completed'])
        
        if self.stats['total_attempts'] > 0:
            accuracy = (self.stats['correct_answers'] / self.stats['total_attempts']) * 100
        else:
            accuracy = 0
        
        print("\nSTATISTIK BELAJAR")
        print("=" * 30)
        print(f"Total Percobaan: {self.stats['total_attempts']}")
        print(f"Jawaban Benar: {self.stats['correct_answers']}")
        print(f"Akurasi: {accuracy:.1f}%")
        print(f"Bab Diselesaikan: {completed_chapters}/{total_chapters}")
        
        if self.stats['last_session']:
            last_session = datetime.fromisoformat(self.stats['last_session'])
            print(f"Sesi Terakhir: {last_session.strftime('%d/%m/%Y %H:%M')}")
        
        # Tampilkan progress per bab
        if self.stats['chapters_completed']:
            print("\nProgress per Bab:")
            for chapter_num, attempts in sorted(self.stats['chapters_completed'].items()):
                chapter_title = self.vocab_data[chapter_num]['title']
                print(f"  Bab {chapter_num}: {attempts} sesi - {chapter_title}")
    
    def proficiency_test(self):
        """Tes kemampuan menyeluruh"""
        print("\nTES KEMAMPUAN MENYELURUH")
        print("=" * 40)
        print("Tes ini akan menguji kosakata dari semua bab")
        print("Siapkan diri Anda!")
        
        input("\nTekan Enter untuk memulai tes...")
        
        all_vocab = []
        for chapter_data in self.vocab_data.values():
            all_vocab.extend([(k, v) for k, v in chapter_data['vocabulary'].items()])
        
        test_size = min(50, len(all_vocab))  # Maksimal 50 pertanyaan
        test_questions = random.sample(all_vocab, test_size)
        
        correct_count = 0
        start_time = time.time()
        
        for i, (num, vocab) in enumerate(test_questions, 1):
            print(f"\nPertanyaan {i}/{test_size}")
            print(f"Korea: {vocab['korean']}")
            
            user_answer = input("Indonesia: ").strip()
            
            if user_answer.lower() == vocab['indonesian'].lower():
                print("✅ Benar!")
                correct_count += 1
            else:
                print(f"❌ Salah! Jawaban: {vocab['indonesian']}")
        
        end_time = time.time()
        test_duration = end_time - start_time
        
        score = (correct_count / test_size) * 100
        average_time = test_duration / test_size
        
        print("\n" + "=" * 40)
        print("HASIL TES KEMAMPUAN")
        print("=" * 40)
        print(f"Skor: {correct_count}/{test_size} ({score:.1f}%)")
        print(f"Durasi Tes: {test_duration:.1f} detik")
        print(f"Rata-rata Waktu per Soal: {average_time:.1f} detik")
        
        # Berikan evaluasi
        if score >= 90:
            print("🎉 Excellent! Penguasaan kosakata sangat baik!")
        elif score >= 75:
            print("👍 Good! Pemahaman kosakata cukup baik!")
        elif score >= 60:
            print("😊 Fair! Perlu lebih banyak latihan!")
        else:
            print("💪 Keep practicing! Anda bisa melakukannya!")
        
        self.stats['total_attempts'] += test_size
        self.stats['correct_answers'] += correct_count
        self.save_stats()
    
    def run(self):
        """Menjalankan program utama"""
        print("Selamat datang di Program Penghafal Kosakata Korea!")
        
        while True:
            self.display_menu()
            choice = input("Pilih menu (1-6): ").strip()
            
            if choice == '1':
                self.study_by_chapter()
            elif choice == '2':
                self.random_quiz()
            elif choice == '3':
                self.review_vocabulary()
            elif choice == '4':
                self.show_stats()
            elif choice == '5':
                self.proficiency_test()
            elif choice == '6':
                print("Terima kasih telah menggunakan program ini!")
                break
            else:
                print("Pilihan tidak valid! Silakan coba lagi.")

# Fungsi untuk mengkonversi data PDF ke format JSON
def create_vocabulary_json():
    """Fungsi untuk membuat file JSON dari data kosakata"""
    # Dalam implementasi nyata, fungsi ini akan membaca file PDF
    # dan mengkonversinya ke format JSON yang terstruktur
    pass

if __name__ == "__main__":
    trainer = VocabularyTrainer()
    trainer.run()
