using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.ComponentModel;
using System.Reflection;

namespace X.Web.MVC.Models
{
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
        public String GetEnumFormattedNamesNoData()
        {
            return "Veri Girilmemiş";
        }
    }
}
