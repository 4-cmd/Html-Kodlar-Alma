# Html-Kodlar-Alma
Bir HTML Sayfasındaki kodları almak
 
  public IActionResult KodAlmak()
 {
     // wwwroot klasör yolunu al
     string wwwrootPath = _webHostEnvironment.WebRootPath;

     // CSV dosyasının tam yolu
     var csvFilePath = Path.Combine(wwwrootPath, "cleaned_bookmarks.csv");

     // Dosya var mı kontrol et
     if (!System.IO.File.Exists(csvFilePath))
         return NotFound("CSV dosyası bulunamadı: " + csvFilePath);

     var driver = new FirefoxDriver();
     driver.Manage().Timeouts().PageLoad = TimeSpan.FromSeconds(30);
     _context.Database.SetCommandTimeout(60);

     using (var streamreader = new StreamReader(csvFilePath)) // CSV dosyasını oku
     {
         using (var csvReader = new CsvReader(streamreader, CultureInfo.InvariantCulture)) // CSV okuyucu oluşur 
         {
             var records = csvReader.GetRecords<dynamic>().ToList(); // Dinamik olarak kayıtları oku

             int satirSayisi = 0; // Satır sayacı

             foreach (var record in records)
             {
                 // var driver = new FirefoxDriver(); // Firefox sürücüsünü başlat

                 satirSayisi++; // Her kayıt için sayacı artır
                 Console.Write("Mevcut Satir " + satirSayisi + ": ");

                 Console.WriteLine($"{record.url}"); // URL'yi yazdır

                 driver.Navigate().GoToUrl(record.url); // URL'ye git

                 try
                 {
                     // Bir işlem yap
                     string pagesource = driver.PageSource; // Sayfa kaynağını al kodları

                     pagesource = pagesource == null ? null : pagesource.Trim(); // Sayfa kaynağını al ve boşlukları temizle

                     var urleklenecek = new UrlofProject
                     {
                         url = record.url, // URL'yi ata
                         htmlofurl = pagesource // Sayfa kaynağını ata
                     };
                     var tümurl = _context.UrlofProjects.Select(i => i.url).ToList(); // Veritabanındaki tüm URL'leri al
                     if(tümurl.Contains(urleklenecek.url))
                     {
                         Console.WriteLine("Bu URL zaten veritabanında mevcut: " + urleklenecek.url);
                         continue; // Eğer URL zaten varsa, bu kaydı atla
                     }
                     _context.UrlofProjects.Add(urleklenecek); // Veritabanına ekle  
                     _context.SaveChanges(); // Değişiklikleri kaydet
                     Console.WriteLine("Veritabanına eklendi: " + record.url);

                     var tags = record.tags.Split(',');

                     foreach (var tag in tags)
                     {
                         var tagekle = new Tags
                         {
                             URLID = urleklenecek.URLID, // UrlofProject ile ilişkilendirme için URLID kullan
                             TagName = tag.Trim() // Etiket adını ata
                         };
                         var tümtag = _context.Tags.Select(i => i.TagName).ToList(); // Veritabanındaki tüm etiketleri al
                         if(tümtag.Contains(tag))
                         {
                             Console.WriteLine("Bu etiket zaten veritabanında mevcut: " + tag.Trim());
                             continue; // Eğer etiket zaten varsa, bu kaydı atla
                         }
                         _context.Tags.Add(tagekle); // Tags tablosuna ekle
                         _context.SaveChanges(); // Değişiklikleri kaydet
                         Console.WriteLine("Etiket eklendi: " + tag.Trim());
                     }


                 }
                 catch (OpenQA.Selenium.NoSuchWindowException ex)
                 {
                     Console.WriteLine("Pencere kapatılmış veya erişilemiyor: " + ex.Message);
                     // Gerekirse driver'ı yeniden başlat
                 }





             }

             Console.WriteLine($"Toplam satır sayısı: {satirSayisi}");
         }
     }

     return Content("Selim");



 }
 
 
