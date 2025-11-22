# Flux Node-RED : Protection Batterie EVCS Victron

Ce flux Node-RED permet de contrôler intelligemment la décharge de la batterie Victron lorsque le chargeur EVCS est en fonctionnement.

## 📦 Versions disponibles

**Version 2 (RECOMMANDÉE)** : `victron-evcs-protection-v2.json`
- ✅ Utilise les **nœuds Victron officiels**
- ✅ Configuration automatique (détection des devices)
- ✅ Plus simple et intégré
- ✅ Monitoring batterie (SOC) inclus
- ✅ Corrigée pour envoyer le bon format au nœud Victron output

**Version 1** : `victron-evcs-protection.json`
- Utilise des nœuds MQTT génériques
- Nécessite configuration manuelle des topics
- Conservée pour compatibilité

## Fonctionnement

### Mode Protection ACTIF ✅
- La batterie continue d'alimenter **uniquement la maison**
- Le chargeur EVCS utilise le **réseau électrique ou le solaire**
- Limite de décharge = consommation maison uniquement

### Mode Normal ⚪
- La batterie peut alimenter **maison + EVCS**
- Fonctionnement standard sans restriction

## Installation

### Prérequis
- **Node-RED** installé sur ton système (ou sur VenusOS)
- **Nœuds Victron** installés dans Node-RED :
  - Palette → Manage palette → Install → `node-red-contrib-victron`
- Connexion au **VenusOS** via le réseau local

### Installation Version 2 (Recommandée - Nœuds Victron)

1. **Ouvrir Node-RED** dans ton navigateur (généralement `http://localhost:1880`)

2. **Importer le flux** :
   - Menu (☰ en haut à droite) → Import
   - Cliquer sur "select a file to import"
   - Sélectionner **`victron-evcs-protection-v2.json`**
   - Cliquer sur "Import"

3. **Configuration automatique** :
   - Les nœuds Victron détectent automatiquement ton VenusOS
   - Si nécessaire, configurer l'adresse VenusOS dans les nœuds Victron (par défaut : `venus.local`)
   - Double-cliquer sur un nœud Victron (vert) → Server → Modifier l'IP si besoin

4. **Vérifier les devices** :
   - Ouvrir chaque nœud Victron (input/output)
   - Vérifier que le bon device est sélectionné (système, EVCS, batterie, etc.)
   - Les devices disponibles apparaissent dans les menus déroulants

5. **Ajuster les valeurs maximales des jauges** (important !) :
   - **Jauge Consommation Maison** :
     - Double-cliquer sur le nœud "Gauge Maison"
     - Modifier "max" selon ta consommation maximale (par défaut : 5000W)
     - Ajuster les seuils "seg1" et "seg2" (par défaut : 1500W et 3000W)
   
   - **Jauge Puissance EVCS** :
     - Double-cliquer sur le nœud "Gauge EVCS"
     - Modifier "max" selon la puissance de ton chargeur (par défaut : 11000W = 11kW)
     - Exemples : 7400W pour 32A monophasé, 22000W pour 32A triphasé
     - Ajuster les seuils selon tes besoins
   
   - **Jauge Batterie** :
     - Par défaut : 0-100% (généralement pas besoin d'ajuster)
     - Seuils : 30% (rouge), 70% (jaune), 100% (vert)

6. **Déployer** :
   - Cliquer sur le bouton rouge "Deploy" en haut à droite

### Installation Version 1 (MQTT générique)

<details>
<summary>Cliquer pour voir les instructions Version 1</summary>

1. **Importer** `victron-evcs-protection.json`

2. **Configurer le broker MQTT** :
   - Double-cliquer sur un nœud MQTT
   - Modifier le serveur : remplacer `venus.local` par l'IP de ton VenusOS
   - Exemple : `192.168.1.100`

3. **Adapter les topics MQTT** selon ton installation :

   | Nœud | Topic par défaut | À adapter |
   |------|------------------|-----------|
   | Consommation maison | `victron/system/Ac/Consumption/L1/Power` | Topic consommation AC |
   | Puissance EVCS | `victron/evcharger/+/Ac/Power` | Topic EVCS |
   | Commande ESS | `W/+/settings/0/Settings/CGwacs/MaxDischargePower` | Topic ESS |

4. **Déployer**

</details>

## Utilisation

> ⚠️ **Important** : Avant la première utilisation, pense à **ajuster les valeurs maximales des jauges** selon ton installation (voir étape 5 de l'installation). Cela permet une meilleure visualisation adaptée à ta configuration.

### Dashboard
Accéder au dashboard : `http://localhost:1880/ui`

**Version 2 - Dashboard complet** :
- 🔒 **Switch "Protection batterie EVCS"** : Active/désactive la protection
- 📊 **Statut** : Affiche le mode actuel et les puissances en temps réel
- 🏠 **Jauge Consommation maison** : Puissance en temps réel de la maison (W)
- ⚡ **Jauge Puissance EVCS** : Puissance en temps réel du chargeur (W)
- 🔋 **Jauge État batterie** : Niveau de charge de la batterie (SOC %)

**Version 1 - Dashboard basique** :
- Switch et jauges de base sans le monitoring batterie

### Exemple concret
```
Situation :
- Maison consomme : 2000W
- EVCS charge à : 7000W
- Total : 9000W

Mode Protection ACTIF :
→ Batterie limitée à 2000W (maison)
→ EVCS prend 7000W du réseau

Mode Normal :
→ Batterie peut fournir 9000W (tout)
```

## Configuration technique

### Version 2 - Nœuds Victron

Les nœuds Victron utilisent l'API D-Bus de VenusOS. Configuration automatique :

**Nœuds utilisés** :
- `victron-input-system` → `/Ac/Consumption/L1/Power` (consommation maison)
- `victron-input-evcharger` → `/Ac/Power` (puissance EVCS)
- `victron-input-battery` → `/Soc` (état de charge batterie)
- `victron-output-settings` → `/Settings/CGwacs/MaxDischargePower` (limite ESS)

Tous les devices disponibles sont détectés automatiquement dans les menus déroulants.

### Version 1 - Topics MQTT (pour référence)

<details>
<summary>Topics MQTT Victron pour la version 1</summary>

Pour trouver tes topics exacts, utilise un client MQTT (ex: MQTT Explorer) :

```
Topics communs Victron :
- N/<portal_id>/system/<device_instance>/Ac/Consumption/L1/Power
- N/<portal_id>/evcharger/<device_instance>/Ac/Power
- W/<portal_id>/settings/0/Settings/CGwacs/MaxDischargePower
```

Remplace `<portal_id>` et `<device_instance>` par tes valeurs.

</details>

## Dépannage

### Version 2 - Problèmes courants

**Les nœuds Victron ne détectent pas les devices**
- Vérifier que Node-RED peut accéder au VenusOS (ping `venus.local` ou IP)
- Vérifier que les nœuds `node-red-contrib-victron` sont bien installés
- Double-cliquer sur un nœud Victron → Server → Vérifier l'adresse IP/hostname
- Redémarrer Node-RED si nécessaire

**La limite de décharge ne s'applique pas**
- Vérifier que l'ESS est activé sur le VenusOS (Menu → ESS)
- Consulter les logs Debug dans Node-RED (panneau de droite)
- Vérifier que le path `/Settings/CGwacs/MaxDischargePower` est accessible
- S'assurer que le mode ESS autorise le contrôle externe

**Nœud Victron en erreur (rouge)**
- Le service correspondant n'existe pas ou n'est pas actif sur VenusOS
- Exemple : pas d'EVCS connecté → nœud `victron-input-evcharger` en erreur
- Vérifier dans VenusOS que le device est bien détecté et actif

### Version 1 - Problèmes courants

**Le flux ne reçoit pas de données**
- Vérifier que le broker MQTT est accessible (ping VenusOS)
- Vérifier les topics avec MQTT Explorer
- Vérifier que VenusOS a le MQTT activé (Settings → Services → MQTT)
- Adapter les topics selon ton `portal_id` et `device_instance`

### Configuration des jauges (recommandé)

**Pourquoi ajuster les valeurs maximales ?**
- Les jauges par défaut sont configurées pour des valeurs génériques
- Pour une meilleure visualisation, adapte-les à ton installation :

| Jauge | Valeur par défaut | À ajuster selon |
|-------|-------------------|-----------------|
| **Consommation Maison** | Max: 5000W | Ta consommation maximale typique |
| **Puissance EVCS** | Max: 11000W | La puissance de ton chargeur |
| **Batterie SOC** | 0-100% | Généralement OK par défaut |

**Exemples de configurations EVCS** :
- Chargeur 16A monophasé (230V) → Max: 3680W
- Chargeur 32A monophasé (230V) → Max: 7400W
- Chargeur 16A triphasé (400V) → Max: 11000W
- Chargeur 32A triphasé (400V) → Max: 22000W

### Améliorations possibles
- Ajouter une persistance en base de données (InfluxDB, SQLite)
- Ajouter des alertes (notifications push, email, Telegram, etc.)
- Ajouter un planificateur (activer/désactiver selon horaires)
- Ajouter une condition sur le SOC (protection uniquement si batterie < X%)
- Ajouter un historique graphique des consommations

## Comparaison des versions

| Critère | Version 2 (Nœuds Victron) | Version 1 (MQTT) |
|---------|---------------------------|------------------|
| **Facilité d'installation** | ⭐⭐⭐⭐⭐ Auto-détection | ⭐⭐⭐ Config manuelle |
| **Configuration** | Automatique | Topics MQTT manuels |
| **Maintenance** | Simple | Peut nécessiter ajustements |
| **Monitoring batterie** | ✅ Inclus (SOC) | ❌ Non inclus |
| **Dépendances** | `node-red-contrib-victron` | Broker MQTT |
| **Recommandation** | ✅ **Recommandée** | Compatible ancien système |

## Support

Pour toute question sur :
- **Nœuds Victron** : https://github.com/victronenergy/node-red-contrib-victron
- **VenusOS / GX** : https://www.victronenergy.com/live/venus-os:start
- **Node-RED** : https://nodered.org/docs/
- **MQTT Victron** : https://github.com/victronenergy/dbus-mqtt

## Fichiers du projet

- `victron-evcs-protection-v2.json` - **Version recommandée** avec nœuds Victron
- `victron-evcs-protection.json` - Version alternative avec MQTT
- `README-Victron-EVCS.md` - Ce fichier (documentation complète)

---

**Note** : Ces flux sont fournis "tel quel". Teste-les en conditions réelles avant de les utiliser de manière permanente. La **Version 2** est recommandée pour sa simplicité et son intégration native avec VenusOS.
