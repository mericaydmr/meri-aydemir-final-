# import java.util.*;
import java.io.*;

public class UcakBiletRezervasyon {

    // --- MODEL SINIFLARI ---

    static class Ucak {
        String model, marka, seriNo;
        int koltukKapasitesi;

        public Ucak(String model, String marka, String seriNo, int kapasite) {
            this.model = model;
            this.marka = marka;
            this.seriNo = seriNo;
            this.koltukKapasitesi = kapasite;
        }

        @Override
        public String toString() {
            return model + " - " + marka + " - Kapasite: " + koltukKapasitesi;
        }
    }

    static class Lokasyon {
        String ulke, sehir, havaalani;
        boolean aktif;

        public Lokasyon(String ulke, String sehir, String havaalani, boolean aktif) {
            this.ulke = ulke;
            this.sehir = sehir;
            this.havaalani = havaalani;
            this.aktif = aktif;
        }

        @Override
        public String toString() {
            return sehir + " (" + havaalani + ") - " + (aktif ? "Aktif" : "Pasif");
        }
    }

    static class Ucus {
        Lokasyon lokasyon;
        String saat;
        Ucak ucak;
        Set<Integer> doluKoltuklar = new HashSet<>();

        public Ucus(Lokasyon lokasyon, String saat, Ucak ucak) {
            this.lokasyon = lokasyon;
            this.saat = saat;
            this.ucak = ucak;
        }

        public boolean yerVarMi() {
            return doluKoltuklar.size() < ucak.koltukKapasitesi;
        }

        public int koltukAyir() {
            for (int i = 1; i <= ucak.koltukKapasitesi; i++) {
                if (!doluKoltuklar.contains(i)) {
                    doluKoltuklar.add(i);
                    return i;
                }
            }
            return -1;
        }

        @Override
        public String toString() {
            return lokasyon.sehir + " - " + saat + " (" + ucak.model + ")";
        }
    }

    static class Rezervasyon {
        String ad, soyad, telefon, tcKimlik;
        int yas, koltukNo;
        Ucus ucus;

        public Rezervasyon(String ad, String soyad, int yas, String telefon, String tcKimlik, Ucus ucus, int koltukNo) {
            this.ad = ad;
            this.soyad = soyad;
            this.yas = yas;
            this.telefon = telefon;
            this.tcKimlik = tcKimlik;
            this.ucus = ucus;
            this.koltukNo = koltukNo;
        }

        @Override
        public String toString() {
            // CSV formati
            return ad + "," + soyad + "," + yas + "," + telefon + "," + tcKimlik + "," +
                   ucus.lokasyon.sehir + "," + ucus.saat + "," + koltukNo;
        }
    }

    // --- SERVIS ---

    static class UcusServisi {
        List<Ucus> ucuslar = new ArrayList<>();
        List<Rezervasyon> rezervasyonlar = new ArrayList<>();

        public UcusServisi() {
            // Lokasyonlar
            Lokasyon l1 = new Lokasyon("Türkiye", "Istanbul", "IST", true);
            Lokasyon l2 = new Lokasyon("Fransa", "Paris", "CDG", true);
            Lokasyon l3 = new Lokasyon("Almanya", "Berlin", "BER", true);

            // Uaklar
            Ucak u1 = new Ucak("A320", "Airbus", "A123", 5);
            Ucak u2 = new Ucak("B737", "Boeing", "B456", 6);

            // Ucuslar
            ucuslar.add(new Ucus(l1, "10:00", u1));
            ucuslar.add(new Ucus(l2, "14:30", u1));
            ucuslar.add(new Ucus(l3, "16:45", u2));
        }

        public void ucuslariListele() {
            for (int i = 0; i < ucuslar.size(); i++) {
                System.out.println((i + 1) + ") " + ucuslar.get(i));
            }
        }

        public boolean rezervasyonYap(int ucusIndex, String ad, String soyad, int yas, String telefon, String tcKimlik) {
            if (ucusIndex < 1 || ucusIndex > ucuslar.size()) return false;
            Ucus ucus = ucuslar.get(ucusIndex - 1);
            if (!ucus.yerVarMi()) return false;
            int koltukNo = ucus.koltukAyir();
            Rezervasyon r = new Rezervasyon(ad, soyad, yas, telefon, tcKimlik, ucus, koltukNo);
            rezervasyonlar.add(r);
            return dosyayaKaydet();
        }

        private boolean dosyayaKaydet() {
            try (PrintWriter pw = new PrintWriter(new FileWriter("rezervasyonlar.csv"))) {
                for (Rezervasyon r : rezervasyonlar) {
                    pw.println(r.toString());
                }
                return true;
            } catch (IOException e) {
                System.out.println("Dosya yazilirken hata: " + e.getMessage());
                return false;
            }
        }
    }

    // --- MAIN ---

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        UcusServisi servis = new UcusServisi();

        while (true) {
            System.out.println("\n=== Ucus Listesi ===");
            servis.ucuslariListele();
            System.out.print("Rezervasyon yapmak istediginiz ucus numarasini girin (0 = cikis): ");
            int secim = -1;

            try {
                secim = Integer.parseInt(sc.nextLine());
            } catch (Exception e) {
                System.out.println("Lütfen gecerli bir sayi girin.");
                continue;
            }

            if (secim == 0) {
                System.out.println("Programdan cikiliyor.");
                break;
            }

            System.out.print("Adiniz: ");
            String ad = sc.nextLine();

            System.out.print("Soyadiniz: ");
            String soyad = sc.nextLine();

            System.out.print("Yasiniz: ");
            int yas = Integer.parseInt(sc.nextLine());

            System.out.print("Telefon Numaraniz: ");
            String telefon = sc.nextLine();

            System.out.print("TC Kimlik Numaraniz: ");
            String tcKimlik = sc.nextLine();

            boolean basarili = servis.rezervasyonYap(secim, ad, soyad, yas, telefon, tcKimlik);

            if (basarili) {
                System.out.println("Rezervasyon basarili! Bilgiler dosyaya kaydedildi.");
            } else {
                System.out.println("Rezervasyon basarisiz! Ucakta yer olmayabilir veya giris bilgileri gecersiz.");
            }
        }

        sc.close();
    }
}
