# Lektion-6-API-Call

## Hämta GBP från API - Nybörjarvänligt sätt

```csharp
		public void Main()
		{
            // try = vi testar denna kod och om något går fel så hamnar i i catch-blocket nedan
            try
            {
                // url = länk till API
                string url = "https://opene.er-api.com/v6/latest/SEK";

                // här hämtar vi resultatet från en GET
                // WebClient = kopplar upp oss mot API
                // DownloadString = hämtar resultatet
                string json = new WebClient().DownloadString(url);

                // Vi vill hitta GBP i vårt JSON
                string searchText = "\"GBP\":";
                // vi får en position i textsträngen
                // position = så här många tecken in i strängen finns ditt resultat
                int start = json.IndexOf(searchText) + searchText.Length;
                int end = json.IndexOf(",", start);

                // extrahera allt som kommer mellan GBP: och nästa kommatecken
                string gbpRateText = json.Substring(start, end - start);

                // System.Globalization.CultureInfo.InvariantCulture = fixar med , eller . i decimaltal
                // i sverige 7,5
                // i programmering 7.5
                double gbpRate = Convert.ToDouble(gbpRateText, System.Globalization.CultureInfo.InvariantCulture);

                Dts.Variables["User::ExchangeRate"].Value = gbpRate;

                // Här skapas en meddelanderuta för att vi ska se värdet
                // kommentera bort den när ni kör
                MessageBox.Show("GBP rate = " + gbpRate.ToString());

                // skickar ut ett resultat som är kopplat till röda eller gröna pilarna i flow
                Dts.TaskResult = (int)ScriptResults.Success;
            }
            // fånga felet från try ovan
            catch (Exception ex)
            {
                // vi använder oss av Exception ex för att hämta message + innerexception. Detta för att underlätta debugging för oss.
                Dts.Variables["User::ErrorMessage"].Value = ex.Message + ex.InnerException;
                Dts.Events.FireError(0, "ExchangeRate", ex.Message.ToString() + ex.InnerException.ToString(), "", 0);
                Dts.TaskResult = (int)ScriptResults.Failure;
            }
        }
```
