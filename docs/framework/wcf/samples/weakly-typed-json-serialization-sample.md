---
title: Weakly-typed JSON Serialization, exemple
ms.custom: 
ms.date: 03/30/2017
ms.prod: .net-framework
ms.reviewer: 
ms.suite: 
ms.technology: dotnet-clr
ms.tgt_pltfrm: 
ms.topic: article
ms.assetid: 0b30e501-4ef5-474d-9fad-a9d559cf9c52
caps.latest.revision: "13"
author: dotnet-bot
ms.author: dotnetcontent
manager: wpickett
ms.workload: dotnet
ms.openlocfilehash: dedf1188afd886c44d897aa1d93ffa226e906ada
ms.sourcegitcommit: 16186c34a957fdd52e5db7294f291f7530ac9d24
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2017
---
# <a name="weakly-typed-json-serialization-sample"></a>Weakly-typed JSON Serialization, exemple
Lors de la sérialisation d'un type défini par l'utilisateur dans un format de transmission donné, ou de la désérialisation d'un format de transmission dans un type défini par l'utilisateur, le type défini par l'utilisateur donné doit être disponible à la fois sur le service et sur le client. En général, l'attribut <xref:System.Runtime.Serialization.DataContractAttribute> est alors appliqué à ces types définis par l'utilisateur, et l'attribut <xref:System.Runtime.Serialization.DataMemberAttribute> est appliqué à leurs membres. Ce mécanisme s’applique également à l’utilisation d’objets JSON (JavaScript Object Notation), comme le décrit la rubrique [How to: Serialize and Deserialize JSON Data](../../../../docs/framework/wcf/feature-details/how-to-serialize-and-deserialize-json-data.md).  
  
 Dans certains scénarios, un service ou un client [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] doit accéder aux objets JSON générés par un service ou un client qui échappe au contrôle du développeur. Comme de plus en plus de services Web exposent publiquement des API JSON, il peut devenir difficile pour le développeur [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] de générer des types définis par l'utilisateur local dans lesquels désérialiser des objets JSON arbitraires. Cet exemple fournit un mécanisme qui permet aux développeurs [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] d'utiliser des objets JSON arbitraires désérialisés, sans créer des types définis par l'utilisateur. C'est ce que l'on appelle la *sérialisation faiblement typée* d'objets JSON, parce que le type dans lequel un objet JSON est désérialisé n'est pas connu au moment de la compilation.  
  
> [!NOTE]
>  La procédure d'installation ainsi que les instructions de génération relatives à cet exemple figurent à la fin de cette rubrique.  
  
 Par exemple, une API de service Web publique retourne l'objet JSON suivant, qui contient des informations à propos d'un utilisateur du service.  
  
```json  
{"personal": {"name": "Paul", "age": 23, "height": 1.7, "isSingle": true, "luckyNumbers": [5,17,21]}, "favoriteBands": ["Band ABC", "Band XYZ"]}  
```  
  
 Pour désérialiser cet objet, un client [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] doit implémenter les types définis par l'utilisateur suivants.  
  
```  
[DataContract]  
 public class MemberProfile  
 {  
     [DataMember]  
     public PersonalInfo personal;  
  
     [DataMember]  
     public string[] favoriteBands;  
 }  
  
 [DataContract]  
 public class PersonalInfo  
 {  
     [DataMember]  
     public string name;  
  
     [DataMember]  
     public int age;  
  
     [DataMember]  
     public double height;  
  
     [DataMember]  
     public bool isSingle;  
  
     [DataMember]  
     public int[] luckyNumbers;  
 }  
```  
  
 Cela peut être gênant, surtout si le client doit gérer plusieurs types d'objets JSON.  
  
 Le type `JsonObject` introduit une représentation faiblement typée de l'objet JSON désérialisé. `JsonObject` repose sur le mappage naturel entre les objets JSON et les dictionnaires [!INCLUDE[dnprdnshort](../../../../includes/dnprdnshort-md.md)] , et le mappage entre les tableaux JSON et les tableaux [!INCLUDE[dnprdnshort](../../../../includes/dnprdnshort-md.md)] . Le code suivant illustre le type `JsonObject` .  
  
```  
// Instantiation of JsonObject json omitted  
  
string name = json["root"]["personal"]["name"];  
int age = json["root"]["personal"]["age"];  
double height = json["root"]["personal"]["height"];  
bool isSingle = json["root"]["personal"]["isSingle"];  
int[] luckyNumbers = {  
                                     json["root"]["personal"]["luckyNumbers"][0],  
                                     json["root"]["personal"]["luckyNumbers"][1],  
                                     json["root"]["personal"]["luckyNumbers"][2]   
                                 };  
string[] favoriteBands = {  
                                        json["root"]["favoriteBands"][0],  
                                        json["root"]["favoriteBands"][1]  
                                    };  
```  
  
 Notez que vous pouvez « parcourir » des objets et tableaux JSON sans avoir besoin de déclarer leur type au moment de la compilation. Pour obtenir une explication de la spécification requise pour l’objet `["root"]` de niveau supérieur, consultez le rubrique [Mapping Between JSON and XML](../../../../docs/framework/wcf/feature-details/mapping-between-json-and-xml.md).  
  
> [!NOTE]
>  La classe `JsonObject` est fournie uniquement à titre d'exemple. Elle n'a pas été testée entièrement et ne doit pas être utilisée dans les environnements de production. L'une des conséquences évidentes de la sérialisation JSON faiblement typée est le manque de sécurité du type lors de l'utilisation de `JsonObject`.  
  
 Pour utiliser le type `JsonObject` , le contrat d'opération cliente doit utiliser <xref:System.ServiceModel.Channels.Message> comme type de retour.  
  
```  
[ServiceContract]  
    interface IClientSideProfileService  
    {  
        // There is no need to write a DataContract for the complex type returned by the service.  
        // The client will use a JsonObject to browse the JSON in the received message.  
  
        [OperationContract]  
        [WebGet(ResponseFormat = WebMessageFormat.Json)]  
        Message GetMemberProfile();  
    }  
```  
  
 Le `JsonObject` est ensuite instancié comme le montre le code suivant.  
  
```  
// Code to instantiate IClientSideProfileService channel omitted…  
  
// Make a request to the service and obtain the Json response  
XmlDictionaryReader reader = channel.GetMemberProfile().GetReaderAtBodyContents();  
  
// Go through the Json as though it is a dictionary. There is no need to map it to a .NET CLR type.  
JsonObject json = new JsonObject(reader);  
```  
  
 Le constructeur `JsonObject` prend un <xref:System.Xml.XmlDictionaryReader>, obtenu par l'intermédiaire de la méthode <xref:System.ServiceModel.Channels.Message.GetReaderAtBodyContents%2A> . Le lecteur contient une représentation XML du message JSON reçu par le client. [!INCLUDE[crdefault](../../../../includes/crdefault-md.md)] la rubrique [Mapping Between JSON and XML](../../../../docs/framework/wcf/feature-details/mapping-between-json-and-xml.md).  
  
 Le programme génère la sortie suivante :  
  
```  
Service listening at http://localhost:8000/.  
To view the JSON output from the sample, navigate to http://localhost:8000/GetMemberProfile  
This is Paul's page. I am 23 years old and I am 1.7 meters tall.  
I am single.  
My lucky numbers are 5, 17, and 21.  
My favorite bands are Band ABC and Band XYZ.  
```  
  
### <a name="to-set-up-build-and-run-the-sample"></a>Pour configurer, générer et exécuter l'exemple  
  
1.  Assurez-vous d’avoir effectué la [procédure d’installation d’à usage unique pour les exemples Windows Communication Foundation](../../../../docs/framework/wcf/samples/one-time-setup-procedure-for-the-wcf-samples.md).  
  
2.  Générez la solution WeaklyTypedJson.sln telle que décrite dans la section [Building the Windows Communication Foundation Samples](../../../../docs/framework/wcf/samples/building-the-samples.md).  
  
3.  Exécutez la solution.  
  
> [!IMPORTANT]
>  Les exemples peuvent déjà être installés sur votre ordinateur. Recherchez le répertoire (par défaut) suivant avant de continuer.  
>   
>  `<InstallDrive>:\WF_WCF_Samples`  
>   
>  Si ce répertoire n’existe pas, accédez à la page [Windows Communication Foundation (WCF) and Windows Workflow Foundation (WF) Samples for .NET Framework 4](http://go.microsoft.com/fwlink/?LinkId=150780) pour télécharger tous les exemples [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] et [!INCLUDE[wf1](../../../../includes/wf1-md.md)] . Cet exemple se trouve dans le répertoire suivant.  
>   
>  `<InstallDrive>:\WF_WCF_Samples\WCF\Scenario\Ajax\WeaklyTypedJson`  
  
## <a name="see-also"></a>Voir aussi
