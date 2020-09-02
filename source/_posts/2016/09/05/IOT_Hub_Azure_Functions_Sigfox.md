---
title: Tutoriel - synchronisation de votre application de fonction avec le concentrateur IoT et exemple de connexion avec un objet Sigfox 
date: 2016-09-05
tags: 
- Azure
- Sigfox
- IoT
- AzureFunctions
---
Ce tutoriel a été rédigé par Eugénie Vinet, stagiaire à la Direction Technique de Microsoft France de avril à août 2016 avec l'aide de son maître de stage Philippe Beraud, Chief Security Advisor à la Direction Technique.
Je remercie Eugénie et Philippe pour cette contribution.

## Synchroniser une application de fonction avec un concentrateur IoT
Pour créer une application de fonction avec Azure Function App, procédez comme suit :

1/	Connectez-vous au portail d’Azure à l’adresse https://portal.azure.com  

2/	Cliquez sur **New** en haut à gauche et cherchez Function App. 

![](/images/160905a/img1.png)

3/ 	Cliquez sur **create**. Entrez ensuite un nom, lié le à un de vos groupes de ressources existants (ou à défaut créez-en un nouveau). Si vous voulez que votre application de fonction fonctionne en continu, choisissez un plan de service B1 associé au minimum.
Le portail **Function App** s’affiche

![](/images/160905a/img2.png)

4/	Cliquez sur **Or create your own custom function en bas. **Une liste des modèles disponibles s’affiche pour votre application de fonction.
![](/images/160905a/img3.png)

5/	Selon votre choix de programmation (C# ou Node) cliquez sur **EventHubTrigger – C#** ou **EventHubTrigger – Node**. 
Azure Function App propose de déclencher une fonction à partir d'un événement dans un concentrateur d'événements, mais pas dans un concentrateur IOT; cependant, un concentrateur IOT peut être vu comme un concentrateur d'événements. C'est ce que nous allons faire. 

6/	Précisez le nom de votre application de fonction et configurez la connexion avec votre concentrateur IoT dans **Event Hub connection**. 
![](/images/160905a/img4.png)

Pour connaître le nom et la chaîne de connexion du concentrateur IoT :

a.	Rendez-vous dans les paramètres de votre concentrateur IoT puis cliquez sur **Messaging**. 

b.	Notez la valeur de **Event-Hub compatible name** (par exemple DecodeSigfoxHub dans notre illustration) ainsi que celle de Compatible endpoint. 
![](/images/160905a/img5.png)

c.	Toujours au niveau de votre concentrateur IoT, allez ensuite dans **Shared Access Policies** et copier la **connection string – primary Key**. Enlevez la partie HostName à la chaine de caractères de manière à ne garder que SharedAccessKeyName et SharedAccessKey.  Vous obtenez par exemple :
`SharedAccessKeyName=iothubowner;SharedAccessKey=MrsSPDzU5=8g2gZZPKXOJ9EJ5RDlo2PVzmPuGniLRwQ=`

d.	Retournez dans la fenêtre de votre application de fonction - ne fermez pas la fenêtre de paramètres de votre concentrateur IoT, vous en aurez besoin plus tard -. Nommez votre application de fonction comme bon vous semble :)

e.	Dans **Event Hub Name**, entrez la valeur précédente de Event-Hub compatible name. Attention cependant, il ne supporte que les noms en minuscule. S’il y a des majuscules dans votre valeur de Event hub compatible name, passez tout en minuscule (par exemple decodesigfoxhub dans notre illustration)  

f.	Cliquez ensuite sur **select** à côté de Event Hub connection. Cliquez ensuite sur** Add a connection** string en haut à droite. 
![](/images/160905a/img6.png)

g.	Dans **Connection name**, spécifiez le nom que vous souhaitez. 

h.	Dans **Connection string**, précisez :
`Endpoint=<CompatibleEndpoint><SharedAccessKeyName><SharedAccessKey>`
Vous obtenez une chaine de caractère semblable à : 
`Endpoint=sb://iothub-ns-decodesigf-499548662a53f6c.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=MrsSPDzU5=8g2gZZPKXOJ9EJ5RDlo2PVzmPuGniLRwQ=`

7/	Validez. 
Votre application de fonction s’exécutera alors à chaque message reçu par l’utilisateur. 
Vous pouvez alors modifier les informations que vous recevez de votre concentrateur IoT pour les renvoyer autre part ou les stocker par exemple.
Vous trouverez ci-après un exemple concret d’usage d’une application de fonction dans notre contexte, à savoir le décodage de trames Sigfox pour les renvoyer sur la solution préconfigurée de surveillance à distance (Cf. portail associé à l’adresse https://www.azureiotsuite.com/) . 

## Exemple avec IoT Suie - Objet Sigfox 
Nous vous proposons dans cette section un exemple de projet utilisant Azure Function App avec un concentrateur IoT.
La solution préconfigurée de surveillance à distance s’attend recevoir des messages sur son concentrateur IoT de type :
```
{
	DeviceId = XX,
	Humidity = XX,
	Temperature = XX,
	ExternalTemperature = XX
}
```
Dans ce projet nous voulons utiliser des objets Sigfox (voir ici http://makers.sigfox.com/#about) qui envoient des trames de type :   
« 422D17 » avec 42 l’identifiant du type d’objet, 2D l’humidité (45%) et 17 la température (en décimal = 23°C). » 
On va supposer que l’humidité et la température sont deux entiers qui vont de 0 à 100. 

### Prérequis
Les prérequis sont les suivants :
•	Avoir lié un device Sigfox qui envoie des données de température et d’humidité, Cf. tutoriel [PUSH YOUR SIGFOX DEVICES DATA TO AZURE IOT HUB](http://danvy.tv/push-your-sigfox-devices-data-to-azure-iot-hub/) .

•	Avoir créé et provisionné une solution préconfigurée de type Solution de surveillance à distance sur le site dédié à l’adresse http://www.azureiotsuite.com/.

•	Avoir suivi l’étape précédente de synchronisation d’une Function App avec un **concentrateur IoT en utilisant le concentrateur IoT Hub lié à l’objet Sigfox**. 

Si ces prérequis sont satisfaits, les étapes de mise en œuvre sont les suivantes :

•	Etape 1 – Ajout de l’objet préconfiguré.

•	Etape 2 – Remplissage du fichier JSON.

•	Etape 3 – Ajout des paquets.

•	Etape 4 – Contenu de votre application de fonction.

Nous les développons dans l’ordre au travers d’une section dédiée propre.

### Etape 1 – Ajout de l’objet préconfiguré
Pour ajouter un objet, procédez comme suit :
1.	Rendez-vous à présent sur l’environnement de démonstration iotmsfrance à l’adresse https://www.azureiotsuite.com/. 
2.	Connectez-vous avec votre compte. Comme cela a été indiqué précédemment, vous devez disposer de privilèges d’administration pour pouvoir enregistrer un objet.
De plus, réalisez cette étape UNIQUEMENT si l’objet n’apparait pas dans la liste des appareils du concentrateur IoT de l’environnement de démonstration (vous pouvez vérifié via le portail Azure ou via l’application **Device Explorer**). Si votre objet est déjà présent dans la liste, il faudra le supprimer en utilisant le **Device Explorer**.
L’application **Device Explorer** est téléchargeable à l’adresse https://github.com/Azure/azure-iot-sdks/releases. Le répertoire d’installation par défaut de cette application est : 
C:\Program Files (x86)\Microsoft\DeviceExplorer
3.	Une fois sur le site, dans la liste des solutions approvisionnées, sélectionnez la **Solution de surveillance à distance**, c.à.d. la solution <VotreSolution>.azurewebsite.net.
4.	Cliquez en bas à gauche sur **Ajouter un appareil** (ou Add a device en anglais). 
![](/images/160905a/img7.png)
L’étape 1 d’ajout d’un appareil s’affiche. 
![](/images/160905a/img8.png)
5.	Dans le cadre **Appareil personnalisé**, sélectionnez **Ajouter un nouveau**. L’étape 2 s’affiche.
![](/images/160905a/img9.png)
6.	Sélectionnez l’option **Me laisser définir mon propre ID d’appareil** et donnez lui le même ID que dans Sigfox /!\ ça ne marchera pas si vous choisissez un id différent.  
7.	Cliquez sur **Vérifier l’ID**. L’ID doit être disponible. 
8.	Cliquez enfin sur **Créer**. 
A ce stade, l’objet n’est pas activé dans la plateforme. Il va falloir modifier les informations relatives à l’objet dans la base NoSQL DocumentDB. Pour cela, connectez-vous au portail d’Azure à l’adresse https://portal.azure.com  et réalisez l’étape suivante.  

## Etape 2 – Remplissage du fichier JSON 
Pour remplir le fichier JSON d’informations lié à l’objet, procédez comme suit :
1.	Rendez-vous dans la rubrique DocumentDB Accounts sur le portail Azure à l’adresse https://portal.azure.com.  ![](/images/160905a/img10.png)
2.	Sélectionner le compte DocumentDB du projet : 
 ![](/images/160905a/img11.png)
3.	Cliquez ensuite sur votre base de données DevMgmtDb (1). Dans **DevMgmtDB**, cliquez sur **Document Explorer** (2).  ![](/images/160905a/img12.png)
Vous devriez voir apparaitre une liste comme celle-ci : 


    {
        "DeviceProperties": {
        "DeviceID": "{NomDeVotreDevice}",
        "HubEnabledState": true,
        "CreatedTime": "0001-01-01T00:00:00",
        "DeviceState": "normal",
        "Manufacturer": "{Manufacturer}",
        "ModelNumber": "{NomDuModelDuDevice}",
        "SerialNumber": "{NumeroDeSérie}",
        "FirmwareVersion": "{FirmwareVersion}",
        "Platform": "{Plateform}",
        "Latitude": {Longitude} ,
        "Longitude": {Latitude},
        "Processor": "{processor}",
        "InstalledRAM": "{installedRam}",
    },
    "SystemProperties": {
        "ICCID": null
    },
     "Commands": [
        {
          "Name": "PingDevice",
          "Parameters": null
        },
        {
          "Name": "StartTelemetry",
          "Parameters": null
        },
    {
      "Name": "StopTelemetry",
      "Parameters": null
    },
    {
      "Name": "ChangeSetPointTemp",
      "Parameters": [
        {
          "Name": "SetPointTemp",
          "Type": "double"
        }
      ]
    },
    {
      "Name": "DiagnosticTelemetry",
      "Parameters": [
        {
          "Name": "Active",
          "Type": "boolean"
        }
      ]
    },
    {
      "Name": "ChangeDeviceState",
      "Parameters": [
        {
          "Name": "DeviceState",
          "Type": "string"
        }
      ]
    }
    ],
    "CommandHistory": [],
    "IsSimulatedDevice": 0,
     "id": "",
     "Telemetry": [
        {
      "Name": "Temperature",
      "DisplayName": "Temperature",
      "Type": "double"
    },
    {
      "Name": "Humidity",
      "DisplayName": "Humidity",
      "Type": "double"
    }
    ],
     "Version": "1.0",
     "ObjectType": "DeviceInfo"
     }
    }

Si vous n’avez pas réalisé l’étape 3 si l’objet existait déjà dans le concentrateur IoT, il est possible qu’il n’y ait aucun fichier relatif à l’objet dans la base de données. Il va alors falloir en créer un en cliquant en haut à gauche. 
Par contre si vous aviez réalisé l’étape 3, le fichier relatif au device devrait se trouver dans la liste. Il devrait avoir un nom de type « 2f23278e-0e82-4850-86f5-b5c3f83a8850 » ou bien le nom du device comme « BFFBA ».  Pour vérifier si un fichier est lié à votre objet, il suffit de l’ouvrir et de lire le DeviceID. Il doit être identique à l’ID que vous avez enregistré sur la page web de la solution de surveillance à distance et identique à l’ID de l’objet enregistré sur le portail Sigfox. (1)
Supprimez ou modifiez le contenu du fichier dont le DeviceID est le nom de votre objet et recopier le texte à droite puis complétez/modifiez les zones en jaune comme suit :

    	HubEnabledState : doit être passé à true.
        DeviceState : doit être normal. 
        CreatedTime : doit être de la forme : AAAA-MM-JJThh:mm:ss 
    Avec AAAA l’année, MM le mois, JJ le jour, hh l’heure, mm les minutes et ss les secondes. 
    Il est recommandé de mettre la date et l’heure qu’il est au moment où vous remplissez le fichier mais cela ne va pas avoir d’impact sur le fonctionnement général de la surveillance à distance. 
        SerialNumber : doit préciser le numéro de série de l’objet si celui en est « équipé ». A défaut, écrivez le nom de l’objet entre guillemets.
        Manufacturer : doit préciser le nom du fabricant de l’objet entre guillemets.
        ModelNumber : doit préciser le numéro de modèle de l’objet si celui en est « équipé ». A défaut, écrivez le nom de l’objet entre guillemets
        FirmwareVersion, Platform, Processor et InstalledRam : si vous ne les connaissez pas pour le device utilisé vous pouvez écrire « None ». 
        Latitude et Longitude : vous avez le choix de son emplacement. L’objet s’affichera à l’endroit indiqué sur la carte. Vous pouvez vous aider de ce site : http://www.latlong.net/ pour trouver la longitude et la latitude d’un endroit précis. Attention : la longitude et la latitude ne doivent pas être écrites entre guillemets sur le document. 
        Id : sera l’ID du document que vous avez créé/modifié. Il est recommandé d’utiliser le nom de l’objet afin de retrouver plus facilement le document. 



## Etape 3 - Ajout des paquets
Tout d’abord nous allons ajouter les paquets nécessaires à notre application de fonction. Pour ceci, nous allons suivre les étapes pour charger un fichier project.json contenant le nom de tous nos paquets (rubrique Package Management sur le site https://azure.microsoft.com/en-us/documentation/articles/functions-reference-csharp/ et File Update ici https://azure.microsoft.com/en-us/documentation/articles/functions-reference/#fileupdate ). 
Procédez comme suit :
1.	Rendez-vous dans les outils de votre application de fonction.
![](/images/160905a/img13.png)
2.	Cliquez sur Kudu puis sur accédez. Vous vous retrouvez donc sur le système de gestion des fichiers de votre application de fonction. 
3.	Allez dans Debug Console puis PowerShell. En utilisant la commande ou bien en utilisant l’explorateur de fichier au-dessus rendez-vous sur D:\home\site\wwwroot\<NomdevotreApplicationDeFonction>>
4.	Ajoutez alors un fichier project.json qui contient les lignes suivantes - un simple glissé-déposé suffit - :  

        {
        "frameworks": {
        "net46":{
            "dependencies": {										        
                    "Microsoft.Azure.Devices.Client":"1.0.6",
	                "Newtonsoft.Json":"8.0.3",				
                    "Microsoft.Azure.Devices":"1.0.6"			
                    }
                }      
            }
        }

## Etape 4 – Définition du contenu de votre application de fonction 
Pour définir le contenu de votre application de fonction, procédez comme suit :

1/	Retournez sur votre application de fonction et ajoutez la classe **DataToReceive**. Le contenu dépendra du contenu du JSON entré pour le callback de votre Device Type dans le portail Sigfox. 
Si vous avez par exemple ce JSON : 


    {
       "device" : "{device}",
       "data" : "{data}",
       "time" : {time},
       "snr" : {snr},
       "station" : "{station}",
       "rssi" : {rssi},
       "seqNumber" : {seqNumber}
    }
Votre classe **DataToReceive** doit ressembler à cela : 
    
    public class DataToReceive
    {
        public string device { get; set; }
        public string data { get; set; }
        public double time { get; set; }
        public string seqNumber { get; set; }
        public string rssi { get; set; }
        public string snr { get; set; }
        public string station { get; set; }
    }

Il faut qu’il y ait au minimum les variables « device » et « data ». 

2/	Créer ensuite une classe **DataToSend** qui réunira les informations que vous voulez envoyer dans votre solution préconfigurée de surveillance à distance. Nous envoyons ici que l’Id de l’objet, la température et l’humidité. Vous pouvez ajouter le paramètre **External Température**. 

```
public class DataToSend
{
    public string DeviceId { get; set; }
    public double Humidity { get; set; }
    public double Temperature { get; set; }
}
```

3/	Créez la classe **SendToIotHub** comme suit. Remplacez bien les valeurs de **connectionString** et d’**iotHubUri** par les éléments relatifs au concentrateur IoT de la solution préconfigurée.  

```
public class SendToIotHub
{
    DeviceClient deviceClient;
    Device device;
    string jsonMessageToSend;
    string connectionString = "{IotHubConnexionString}";
    string iotHubUri = "{IotHubName}.azure-devices.net";

    public async void sendData(DataToSend data, TraceWriter log)
    {
        RegistryManager registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        device = new Device(data.DeviceId);
        device = await AddDeviceAsync1(data.DeviceId, registryManager);
        deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey(data.DeviceId,device.Authentication.SymmetricKey.PrimaryKey.ToString()));
       jsonMessageToSend = JsonConvert.SerializeObject(data);
        var messageToSend = new Microsoft.Azure.Devices.Client.Message(System.Text.Encoding.ASCII.GetBytes(jsonMessageToSend));

        deviceClient.SendEventAsync(messageToSend);
        log.Info($"C# Event Hub trigger function processed a message: {jsonMessageToSend}");
    }

        private async Task<Device> AddDeviceAsync1(string deviceId, RegistryManager registryManager)
        {
            Device device;
            try
            {
                device = await registryManager.AddDeviceAsync(new Device(deviceId));
    
            }
            catch (DeviceAlreadyExistsException)
            {
                device = await registryManager.GetDeviceAsync(deviceId);
            }
            return device;
        }
    }
```

4/	Ajoutez les directives using « de circonstance » au début de votre application de fonction :

```        
using Microsoft.Azure.Devices;
using Microsoft.Azure.Devices.Client;
using System;
using Microsoft.Azure.Devices.Common.Exceptions;
using Newtonsoft.Json;
```

5/	Complétez maintenant la méthode **Run** : 

```
public static void Run(string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessage}");

    DataToReceive receivedData = new DataToReceive();
    DataToSend sentData = new DataToSend();
    SendToIotHub sendDataToIotHub = new SendToIotHub();
    receivedData = JsonConvert.DeserializeObject<DataToReceive>(myEventHubMessage);

    if (receivedData.data != null && receivedData.data.Substring(0, 2) == "42")
    {
            string data = receivedData.data;
            sentData.Humidity = Convert.ToInt32(data.Substring(2, 2), 16);
            sentData.Temperature = Convert.ToInt32(data.Substring(4, 2), 16);
            sentData.DeviceId = receivedData.device;
            sendDataToIotHub.sendData(sentData, log);
    }
}
```

Vous obtiendrez quelque chose de similaire à la capture qui suit si tout va bien :

![](/images/160905a/img14.png)

Vous devriez voir dans les logs les messages reçus et envoyés si vous gardez les lignes de code `log.info (…..)`. 

Bien sûr, nous partons du principe que les trames à utiliser sont de la forme **« 422D17 »** (comme développé précédemment) mais il est possible que vous ayez à faire des calculs sur chaque élément de votre trame pour obtenir les valeurs décimales renvoyées par vos capteurs. Pour ceci, renseignez-vous auprès du constructeur de l’objet.  


Et voilà ! 
Ceci conclut ce tutoriel.

