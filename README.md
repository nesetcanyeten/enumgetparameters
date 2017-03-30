# C# Dilinde Enumaration (Enum) Yapıları Kullanarak Kod Yükünden Kurtulma

![metin](https://www.google.com.tr/url?sa=i&rct=j&q=&esrc=s&source=images&cd=&cad=rja&uact=8&ved=0ahUKEwiu2u-L1v3SAhVEXhoKHeDwCFoQjRwIBw&url=https%3A%2F%2Fwww.takasbank.com.tr%2Ftr%2FSayfalar%2FNumaralandirma.aspx&bvm=bv.151426398,bs.2,d.bGg&psig=AFQjCNEHOhBSVx42lzSFeWMR80wtYKWD6Q&ust=1490944025919899)

Enumaration ya da Enum, yazılımda kullanılacak değişkenlerin alabileceği değerlerin her zaman aynı olduğu durumlarda kullanılır. Yazılan kodun daha okunaklı ve düzenli olmasını sağlar. En genel haliyle "enum" yapısı aşağıdaki gibi ifade edilir:


    public enum Bitki
    {
        Bugday = 1, Misir = 2, Arpa = 3, Aycicegi = 4, Mercimek = 5
    }

    string[] Bitkiler = Enum.Getnames(typeof(Bitki));


Bu kod ‘Bitki’ isimli "enum"daki sabitleri yani tanımlanan bitkileri ‘Bitkiler’ adlı diziye atar.

Yazdığımız kodda sık kullandığımız ifadelere belirli numaralar vererek ihtiyaç duyduğumuzda çağırmak ya da veri kaydı sırasında uzun ifadeler yerine onu tanımlayan eden numarayı kaydetmek mantıklı bir davranış olacaktır. Kaydettiğimiz sayının tanımladığı ifadeyi "enum" yapısı kullanarak çağırabiliriz. Aşağıda tanımladığımız "enum" türü bu amaçla oluşturulmuştur.

        public enum Durum
        {
            [Description("Hurdaya Ayrıldı.")]
            hurdayaAyrildi = 10,
            [Description("Bakım Yapılıyor.")]
            bakimYapiliyor = 20,
            [Description("Tamir Ediliyor.")]
            tamirEdiliyor = 30,
            [Description("Sorun Çözüldü.")]
            sorunCozuldu = 40,
            [Description("Ürün Teslim Edildi.")]
            urunTeslimEdildi = 50
        }

Veritabanımızda, "Durum" "enum"ındaki cümleler yerine bu cümleleri ifade eden sayıları sakladığımızı ve bu sayıları kullanarak ürünlerimizin durumlarını bir tabloda göstereceğimizi düşünelim. Sayıları kullanarak ürün durumlarına ulaşabilmek için önerdiğimiz yapı aşağıda verilmiştir.

    public class EnumGetParameters
    {
        public EnumGetParameters() { }
        public string GetEnumDescription(Enum currentEnum)
            {
                string description = String.Empty;
                DescriptionAttribute da;

                FieldInfo fi = currentEnum.GetType().
                            GetField(currentEnum.ToString());

                da = (DescriptionAttribute)Attribute.GetCustomAttribute(fi,
                            typeof(DescriptionAttribute));
                if (da != null)
                    description = da.Description;
                else
                    description = currentEnum.ToString();

                return description;
            }

        public Dictionary<string, string> GetEnumFormattedNames<TEnum>()
            {
                var enumType = typeof(TEnum);
                if (enumType == typeof(Enum))
                    throw new ArgumentException("typeof(TEnum) == System.Enum", "TEnum");

                if (!(enumType.IsEnum))
                    throw new ArgumentException(String.Format("typeof({0}).IsEnum == false", enumType), "TEnum");

                FieldInfo[] fields = enumType.GetFields();
                Dictionary<string, string> forattedValues = new Dictionary<string, string>();
                foreach (var field in fields)
                {
                    if (field.Name.Equals("value__")) continue;
                    var fieldValue = field.GetRawConstantValue();

                    forattedValues.Add(field.Name.ToString(), fieldValue.ToString());
                }

                Dictionary<string, string> forattedNames = new Dictionary<string, string>();
                var vrs = Enum.GetValues(typeof(TEnum));
                var list = Enum.GetValues(enumType).OfType<TEnum>().ToList<TEnum>();

                foreach (TEnum item in list)
                {
                    forattedNames.Add(forattedValues[item.ToString()], GetEnumDescription(item as Enum));
                }

                return forattedNames;
            }
    }

Yukarıda oluşturduğumuz metodu aşağıdaki gibi çağırırız. Çağırdığımız "enum"daki tüm veriler durumlar değişkenine doldurulur. Veritabanından çekilen her ürün için sayı halindeki UrunDurumu değişkenine "enum"ımızdaki cümleler atanır.

    var durumlar = enumGetParameters.GetEnumFormattedNames<X.Data.XDatabase.Durum>();

    foreach (var item in urunler)
    {
        item.UrunDurumu = item.UrunDurumu != null ? durumlar[item.UrunDurumu] : "Bilinmiyor";
    }

"Enum"daki değerlerin "03", "020" veya "00" şeklinde olduğunu varsayalım. Bu durumda değerler:

    var durum_ = item.UrunDurumu != null ? durumlar[Convert.ToInt32(item.UrunDurumu).ToString()] : "Bilinmiyor";
    item.UrunDurumu = durum_ ;

şeklinde atanır.

Bahsettiğimiz metot olmasaydı aynı işi yapabilmek için aşağıdaki "if" kontrollerini yazmamız gerekecekti.

    foreach (var item in urunler)
    {
        if(item.UrunDurumu == "10")
        {
            item.UrunDurumu = "Hurdaya Ayrıldı."
        }
        else if (item.UrunDurumu == "20")
        {
            item.UrunDurumu = "Bakım Yapılıyor."
        }
        else if (item.UrunDurumu == "30")
        {
            item.UrunDurumu = "Tamir Ediliyor."
        }
        else if (item.UrunDurumu == "40")
        {
            item.UrunDurumu = "Sorun Çözüldü."
        }
        else if (item.UrunDurumu == "50")
        {
            item.UrunDurumu = "Ürün Teslim Edildi."
        }
        else
        {
            item.UrunDurumu = "Bilinmiyor."
        }
    }

Kod parçasından da anlaşılacağı gibi yüksek miktarda farklı değer için yüzlerce defa "if" kontrolü yazmak zorunda kalacaktık. Veritabanından gelen yüksek miktardaki veriyi de düşünürsek önerdiğimiz metot bizi ağır bir yükten kurtarmaktadır.
