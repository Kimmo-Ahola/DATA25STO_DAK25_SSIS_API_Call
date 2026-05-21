# Lektion-6-API-Call

## Hämta GBP från API - Nybörjarvänligt sätt

```csharp
		public void Main()
		{
            // try = vi testar denna kod och om något går fel så hamnar i i catch-blocket nedan
            try
            {
                // url = länk till API
                string url = "https://open.er-api.com/v6/latest/SEK";

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

## Ett "bättre" sätt är att skapa klasser i C# som matchar din JSON.

1. kopiera din rådata, till exempel:
```json
{"result":"success","provider":"https://www.exchangerate-api.com","documentation":"https://www.exchangerate-api.com/docs/free","terms_of_use":"https://www.exchangerate-api.com/terms","time_last_update_unix":1779321751,"time_last_update_utc":"Thu, 21 May 2026 00:02:31 +0000","time_next_update_unix":1779409181,"time_next_update_utc":"Fri, 22 May 2026 00:19:41 +0000","time_eol_unix":0,"base_code":"SEK","rates":{"SEK":1,"AED":0.392505,"AFN":6.734705,"ALL":8.759945,"AMD":39.183549,"ANG":0.191309,"AOA":100.313632,"ARS":149.502905,"AUD":0.149669,"AWG":0.191309,"AZN":0.180899,"BAM":0.179999,"BBD":0.213754,"BDT":13.052725,"BGN":0.179999,"BHD":0.040186,"BIF":316.679612,"BMD":0.106877,"BND":0.136644,"BOB":0.736588,"BRL":0.536655,"BSD":0.106877,"BTN":10.343207,"BWP":1.503834,"BYN":0.291692,"BZD":0.213754,"CAD":0.147004,"CDF":243.41791,"CHF":0.084244,"CLF":0.002442,"CLP":96.52849,"CNH":0.726608,"CNY":0.726761,"COP":403.200672,"CRC":48.162299,"CUP":2.565042,"CVE":10.147917,"CZK":2.236635,"DJF":18.994244,"DKK":0.686685,"DOP":6.261451,"DZD":14.1184,"EGP":5.670578,"ERN":1.603151,"ETB":16.77018,"EUR":0.092032,"FJD":0.234834,"FKP":0.079629,"FOK":0.686668,"GBP":0.079629,"GEL":0.284299,"GGP":0.079629,"GHS":1.225791,"GIP":0.079629,"GMD":7.896339,"GNF":932.905623,"GTQ":0.810972,"GYD":22.234492,"HKD":0.836854,"HNL":2.83006,"HRK":0.693416,"HTG":13.915529,"HUF":33.126267,"IDR":1884.532907,"ILS":0.310702,"IMP":0.079629,"INR":10.34327,"IQD":139.393162,"IRR":139855.151159,"ISK":13.171801,"JEP":0.079629,"JMD":16.833903,"JOD":0.075776,"JPY":16.981718,"KES":13.778377,"KGS":9.308929,"KHR":429.184211,"KID":0.149681,"KMF":45.276816,"KRW":160.448856,"KWD":0.032807,"KYD":0.089064,"KZT":50.356371,"LAK":2334.260403,"LBP":9565.46968,"LKR":35.711573,"LRD":19.488404,"LSL":1.763592,"LYD":0.677228,"MAD":0.986128,"MDL":1.840516,"MGA":446.821918,"MKD":5.6556,"MMK":223.485655,"MNT":381.601245,"MOP":0.8615,"MRU":4.262119,"MUR":5.044266,"MVR":1.643738,"MWK":185.515954,"MXN":1.853038,"MYR":0.423415,"MZN":6.787874,"NAD":1.763592,"NGN":145.928802,"NIO":3.915812,"NOK":0.990628,"NPR":16.549132,"NZD":0.182413,"OMR":0.041094,"PAB":0.106877,"PEN":0.365205,"PGK":0.466873,"PHP":6.579826,"PKR":29.795044,"PLN":0.390963,"PYG":651.095546,"QAR":0.389031,"RON":0.480957,"RSD":10.79816,"RUB":7.58754,"RWF":156.057102,"SAR":0.400788,"SBD":0.852905,"SCR":1.575859,"SDG":47.548105,"SGD":0.136644,"SHP":0.079629,"SLE":2.601405,"SLL":2600.674474,"SOS":60.741155,"SRD":3.955135,"SSP":502.407378,"STN":2.254786,"SYP":11.948458,"SZL":1.763592,"THB":3.484799,"TJS":0.990161,"TMT":0.372611,"TND":0.310474,"TOP":0.254703,"TRY":4.879931,"TTD":0.721733,"TVD":0.149681,"TWD":3.373452,"TZS":279.961778,"UAH":4.706245,"UGX":402.685502,"USD":0.106903,"UYU":4.289592,"UZS":1293.599417,"VES":55.982369,"VND":2765.299626,"VUV":12.684932,"WST":0.287562,"XAF":60.369088,"XCD":0.288567,"XCG":0.191309,"XDR":0.07802,"XOF":60.369088,"XPF":10.982372,"YER":25.390471,"ZAR":1.763596,"ZMW":2.014551,"ZWG":2.802623,"ZWL":2.803411}}
```
1. Högerklicka på ditt projektnamn och skapa en ny klass genom add köra add > class
<img width="603" height="626" alt="image" src="https://github.com/user-attachments/assets/4b6699fa-cf4d-47bc-983c-58644e54b8e0" />

1. Inuti din nya klassfil kan du klistra in (och rensa öonskad data) från din klass via smart paste
<img width="449" height="366" alt="image" src="https://github.com/user-attachments/assets/0c8af90a-29d2-442b-ad77-e952c693a811" />


```csharp
public class MoneyData
{
    public Rates rates { get; set; }
}

public class Rates
{
    public int SEK { get; set; }
    public float AED { get; set; }
    public float AFN { get; set; }
    public float ALL { get; set; }
    public float AMD { get; set; }
    public float ANG { get; set; }
    public float AOA { get; set; }
    public float ARS { get; set; }
    public float AUD { get; set; }
    public float AWG { get; set; }
    public float AZN { get; set; }
    public float BAM { get; set; }
    public float BBD { get; set; }
    public float BDT { get; set; }
    public float BGN { get; set; }
    public float BHD { get; set; }
    public float BIF { get; set; }
    public float BMD { get; set; }
    public float BND { get; set; }
    public float BOB { get; set; }
    public float BRL { get; set; }
    public float BSD { get; set; }
    public float BTN { get; set; }
    public float BWP { get; set; }
    public float BYN { get; set; }
    public float BZD { get; set; }
    public float CAD { get; set; }
    public float CDF { get; set; }
    public float CHF { get; set; }
    public float CLF { get; set; }
    public float CLP { get; set; }
    public float CNH { get; set; }
    public float CNY { get; set; }
    public float COP { get; set; }
    public float CRC { get; set; }
    public float CUP { get; set; }
    public float CVE { get; set; }
    public float CZK { get; set; }
    public float DJF { get; set; }
    public float DKK { get; set; }
    public float DOP { get; set; }
    public float DZD { get; set; }
    public float EGP { get; set; }
    public float ERN { get; set; }
    public float ETB { get; set; }
    public float EUR { get; set; }
    public float FJD { get; set; }
    public float FKP { get; set; }
    public float FOK { get; set; }
    public float GBP { get; set; }
    public float GEL { get; set; }
    public float GGP { get; set; }
    public float GHS { get; set; }
    public float GIP { get; set; }
    public float GMD { get; set; }
    public float GNF { get; set; }
    public float GTQ { get; set; }
    public float GYD { get; set; }
    public float HKD { get; set; }
    public float HNL { get; set; }
    public float HRK { get; set; }
    public float HTG { get; set; }
    public float HUF { get; set; }
    public float IDR { get; set; }
    public float ILS { get; set; }
    public float IMP { get; set; }
    public float INR { get; set; }
    public float IQD { get; set; }
    public float IRR { get; set; }
    public float ISK { get; set; }
    public float JEP { get; set; }
    public float JMD { get; set; }
    public float JOD { get; set; }
    public float JPY { get; set; }
    public float KES { get; set; }
    public float KGS { get; set; }
    public float KHR { get; set; }
    public float KID { get; set; }
    public float KMF { get; set; }
    public float KRW { get; set; }
    public float KWD { get; set; }
    public float KYD { get; set; }
    public float KZT { get; set; }
    public float LAK { get; set; }
    public float LBP { get; set; }
    public float LKR { get; set; }
    public float LRD { get; set; }
    public float LSL { get; set; }
    public float LYD { get; set; }
    public float MAD { get; set; }
    public float MDL { get; set; }
    public float MGA { get; set; }
    public float MKD { get; set; }
    public float MMK { get; set; }
    public float MNT { get; set; }
    public float MOP { get; set; }
    public float MRU { get; set; }
    public float MUR { get; set; }
    public float MVR { get; set; }
    public float MWK { get; set; }
    public float MXN { get; set; }
    public float MYR { get; set; }
    public float MZN { get; set; }
    public float NAD { get; set; }
    public float NGN { get; set; }
    public float NIO { get; set; }
    public float NOK { get; set; }
    public float NPR { get; set; }
    public float NZD { get; set; }
    public float OMR { get; set; }
    public float PAB { get; set; }
    public float PEN { get; set; }
    public float PGK { get; set; }
    public float PHP { get; set; }
    public float PKR { get; set; }
    public float PLN { get; set; }
    public float PYG { get; set; }
    public float QAR { get; set; }
    public float RON { get; set; }
    public float RSD { get; set; }
    public float RUB { get; set; }
    public float RWF { get; set; }
    public float SAR { get; set; }
    public float SBD { get; set; }
    public float SCR { get; set; }
    public float SDG { get; set; }
    public float SGD { get; set; }
    public float SHP { get; set; }
    public float SLE { get; set; }
    public float SLL { get; set; }
    public float SOS { get; set; }
    public float SRD { get; set; }
    public float SSP { get; set; }
    public float STN { get; set; }
    public float SYP { get; set; }
    public float SZL { get; set; }
    public float THB { get; set; }
    public float TJS { get; set; }
    public float TMT { get; set; }
    public float TND { get; set; }
    public float TOP { get; set; }
    public float TRY { get; set; }
    public float TTD { get; set; }
    public float TVD { get; set; }
    public float TWD { get; set; }
    public float TZS { get; set; }
    public float UAH { get; set; }
    public float UGX { get; set; }
    public float USD { get; set; }
    public float UYU { get; set; }
    public float UZS { get; set; }
    public float VES { get; set; }
    public float VND { get; set; }
    public float VUV { get; set; }
    public float WST { get; set; }
    public float XAF { get; set; }
    public float XCD { get; set; }
    public float XCG { get; set; }
    public float XDR { get; set; }
    public float XOF { get; set; }
    public float XPF { get; set; }
    public float YER { get; set; }
    public float ZAR { get; set; }
    public float ZMW { get; set; }
    public float ZWG { get; set; }
    public float ZWL { get; set; }
}
```

1. Välj Nuget Package Manager
<img width="574" height="340" alt="image" src="https://github.com/user-attachments/assets/ffca98cb-968f-4fbf-946e-190c6fa911f9" />

1. Installera Newtonsoft.JSON
<img width="1249" height="405" alt="image" src="https://github.com/user-attachments/assets/3b577f9a-45c9-458e-91a0-c4b70a5c72f9" />

1. Vår Main blir nu som nedan:

```csharp
public void Main()
{
    // try = vi testar denna kod och om något går fel så hamnar i i catch-blocket nedan
    try
    {
        string url = "https://opene.er-api.com/v6/latest/SEK";
        string json = new WebClient().DownloadString(url);

		// Använd JsonConvert.Deserializer här för att omvandla vår text till riktiga klasser
        MoneyData moneyData = JsonConvert.DeserializeObject<MoneyData>(json);
        double GBP = moneyData.rates.GBP;
        Dts.Variables["User::ExchangeRate"].Value = GBP;
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
