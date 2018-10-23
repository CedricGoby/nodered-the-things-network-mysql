# nodered-the-things-network-mysql.json

Cet exemple de **flow node-red** permet de récupérer les **données d'un capteur** via le réseau **LoRa "The Things Network"** puis d'insérer ces données dans une base de données **MySQL**.

Ce flow récupère une mesure (une température) puis l'insère dans le champs "*temperature*" de la table "*my_node*" présente dans la base "*my_ttn_application*" . La date et l'heure de transmission de la mesure sont insérées dans le même enregistrement dans le champs "*date*" de la table "*my_node*"

## Architecture

Ci-dessous le circuit emprunté par les données depuis le capteur vers la base de données MySQL dès qu'une mesure est prise par le capteur :

  * Le node* envoit un message (la donnée du capteur et des métas-données) sur le réseau LoRa
  * La gateway** LoRa réceptionne le message
  * La gateway transmet le message vers le serveur d'application "The Things Network"
  * Node-red est connecté au serveur d'application "The Things Network" avec le prorocole mqtts et récupère le message
  * Node-red insère les données du message dans la base de données MySQL

\*node (noeud) = capteur + arduino + puce LoRa
**Gateway (passerelle) = Réception des données LoRa émises par le node, transfert des données du nodes vers le serveur d'application "The Things Network"

## Prérequis

### Node-red

Node-red doit être installé : [https://nodered.org/docs/getting-started/installation](https://nodered.org/docs/getting-started/installation) 

### Encodage pour l'envoi des données (Arduino)

  * Les données envoyées par le node sont encodées en hexadécimal. Ci-dessous un exemple pour encoder le contenu de la variable "*msg*" dans un croquis arduino avant son envoi via le réseau LoRa.

```
	for (unsigned int i = 0; i < msg.length(); i++) {
	    Serial.print(msg[i] >> 4, HEX);
	    Serial.print(msg[i] & 0xF, HEX);
	    Serial.print(" ");
	  }
```

### The Things Network

  * Ouvrir un compte sur "The Things Network" : [https://www.thethingsnetwork.org/](https://www.thethingsnetwork.org/)
  * Depuis la console "The Things Network" ([https://console.thethingsnetwork.org/](https://console.thethingsnetwork.org/) ) :
      * Créer une application (ex: my_ttn_application)
      * Enregistrer le node dans l'application et lui donner un nom (ex: my_node)
      * Dans Applications > my_ttn_application > Payload Formats utiliser le decoder suivant :

```
function Decoder(bytes, port) {
  // Decode an uplink message from a buffer
  // (array) of bytes to an object of fields.
  var decoded = {};

  // if (port === 1) decoded.led = bytes[0];

  return decoded;
}
```

### MySQL

MySQL est installée sur la même machine que node-red.

  * Créer une base de données "*my_ttn_application*"
  * Créer une table "*my_node*"
  * Créer un utilisateur "nodered" avec les privilèges *SELECT,INSERT,UPDATE* sur la base de données "*my_ttn_application*"

Ci-dessous la commande SQL pour créer la base *my_ttn_application* et la table *my_node*

```sql
-- phpMyAdmin SQL Dump

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Base de données :  `my_ttn_application`
--

-- --------------------------------------------------------

--
-- Structure de la table `my_node`
--

CREATE TABLE `my_node` (
  `id` int(11) NOT NULL,
  `date` datetime(6) NOT NULL,
  `temperature` varchar(6) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

--
-- Index pour les tables exportées
--

--
-- Index pour la table `my_node`
--
ALTER TABLE `my_node`
  ADD PRIMARY KEY (`id`);

--
-- AUTO_INCREMENT pour les tables exportées
--

--
-- AUTO_INCREMENT pour la table `my_node`
--
ALTER TABLE `my_node`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
```

### Installer le certificat MQTT "The Things Network" dans Node-red

Ce certificat permet d'utiliser mqtts (mqtt over SSL) avec "The Things Network"
 
  * Télécharger le certificat "The Things Network" : [https://console.thethingsnetwork.org/mqtt-ca.pem](https://console.thethingsnetwork.org/mqtt-ca.pem)  
  * Copier le certificat sur la machine qui héberge node-red, dans le home de l'utilisateur qui lance le service node-red (jonas dans l'exemple ci-dessous)
  
    /home/jonas/mqtt-ca.pem

### Modules utilisés dans node-red

  * **mqtt** node : Récupère les données de l'objet connecté depuis "The Things Network"
  * **json** node : Convertit les données au format JSON 
  * **function** node : Construit la requête SQL pour insérer les données dans la base de données
  * **mysql** node : Gère la connexion à la base de données et l'execution de la requête SQL
  * **debug** node : Affiche la requête SQL transmise et le résultat de l'execution

### Ouverture de port

Le port **TCP  8883** doit être ouvert depuis la machine qui héberge node-red afin d'utiliser **mqtts** (mqtt over SSL) avec "The Things Network".

#### Licence
GNU General Public License v3.0