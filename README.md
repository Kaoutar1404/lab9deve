# 🌸 Gestion des Étudiants — Application Android & Backend PHP

<div align="center">

**✨ Réalisé par Kaoutar Menacera ✨**

*Travail Pratique · Intégration Mobile · Android + PHP + MySQL*

![PHP](https://img.shields.io/badge/PHP-8-blueviolet?style=flat-square)
![Android](https://img.shields.io/badge/Android-Java-pink?style=flat-square)
![MySQL](https://img.shields.io/badge/MySQL-XAMPP-ff69b4?style=flat-square)
![Volley](https://img.shields.io/badge/HTTP-Volley-orchid?style=flat-square)

</div>

---

## 🌺 Présentation du projet

Ce travail pratique a pour objectif de concevoir une application mobile Android communiquant avec un serveur web PHP pour gérer une liste d'étudiants. Les données circulent au format **JSON** entre les deux côtés, désérialisées grâce à **Gson** et acheminées via **Volley**.

### Ce que ce projet accomplit

| Composante | Rôle |
|---|---|
| 🗄️ MySQL (XAMPP) | Stockage des données étudiants |
| 🐘 PHP + PDO | Exposition de services REST (GET / POST) |
| 📱 Android Studio | Interface mobile de saisie et consultation |
| 🌐 Volley | Client HTTP pour les appels réseau |
| 🔍 Gson | Désérialisation du JSON en objets Java |

---

## 🗄️ Partie 1 — Base de données MySQL

### Pré-requis
XAMPP installé avec **Apache** et **MySQL** activés.

### Mise en place

Ouvrir **phpMyAdmin** à l'adresse `http://localhost/phpmyadmin` et exécuter :

```sql
-- 1. Créer la base de données
CREATE DATABASE school1;
USE school1;

-- 2. Créer la table des étudiants
CREATE TABLE Etudiant (
  id     INT AUTO_INCREMENT PRIMARY KEY,
  nom    VARCHAR(50),
  prenom VARCHAR(50),
  ville  VARCHAR(50),
  sexe   VARCHAR(10)
);

-- 3. Insérer des données de test
INSERT INTO Etudiant (nom, prenom, ville, sexe)
VALUES
  ('Lachgar', 'Mohamed', 'Rabat',     'homme'),
  ('Safi',    'Amine',   'Marrakech', 'homme');
```
<img width="959" height="499" alt="xamp" src="https://github.com/user-attachments/assets/75af52ae-9039-45d3-adfa-82ef74628f5a" />



https://github.com/user-attachments/assets/583ac7ec-0242-433f-ab94-2ac3d0f3f6f3

<img width="811" height="242" alt="database" src="https://github.com/user-attachments/assets/bb3f9a84-242a-4e5f-a982-b35d19d81ac8" />



> 💜 La table `Etudiant` stocke : un identifiant auto-incrémenté, le nom, le prénom, la ville et le sexe.

---

## 🐘 Partie 2 — Service Web PHP

### Organisation des fichiers

Tout le projet PHP se trouve dans `C:\xampp\htdocs\projet\` :

```
projet/
│
├── 📁 classes/
│   └── Etudiant.php         ← Modèle métier
│
├── 📁 connexion/
│   └── Connexion.php        ← Accès PDO à la base
│
├── 📁 dao/
│   └── IDao.php             ← Contrat CRUD
│
├── 📁 service/
│   └── EtudiantService.php  ← Implémentation des requêtes
│
└── 📁 ws/
    ├── createEtudiant.php   ← Route POST
    └── loadEtudiant.php     ← Route GET
```

---

### 💾 `connexion/Connexion.php` — Connexion PDO

```php
<?php
class Connexion {
    private $connexion;

    public function __construct() {
        try {
            $this->connexion = new PDO(
                "mysql:host=localhost;dbname=school1;charset=utf8",
                "root",
                ""
            );
            $this->connexion->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            die('Erreur de connexion : ' . $e->getMessage());
        }
    }

    public function getConnexion() {
        return $this->connexion;
    }
}
?>
```

---

### 👤 `classes/Etudiant.php` — Entité Etudiant

```php
<?php
class Etudiant {
    private $id, $nom, $prenom, $ville, $sexe;

    public function __construct($id, $nom, $prenom, $ville, $sexe) {
        $this->id     = $id;
        $this->nom    = $nom;
        $this->prenom = $prenom;
        $this->ville  = $ville;
        $this->sexe   = $sexe;
    }

    public function getNom()    { return $this->nom;    }
    public function getPrenom() { return $this->prenom; }
    public function getVille()  { return $this->ville;  }
    public function getSexe()   { return $this->sexe;   }
}
?>
```

---

### 📋 `dao/IDao.php` — Interface d'accès aux données

```php
<?php
interface IDao {
    function create($o);
    function delete($o);
    function update($o);
    function findAll();
    function findById($id);
}
?>
```

---

### ⚙️ `service/EtudiantService.php` — Logique métier

```php
<?php
include_once '../classes/Etudiant.php';
include_once '../connexion/Connexion.php';
include_once '../dao/IDao.php';

class EtudiantService implements IDao {
    private $connexion;

    public function __construct() {
        $this->connexion = new Connexion();
    }

    public function create($o) {
        $sql  = "INSERT INTO Etudiant (nom, prenom, ville, sexe)
                 VALUES (:nom, :prenom, :ville, :sexe)";
        $stmt = $this->connexion->getConnexion()->prepare($sql);
        $stmt->execute([
            ':nom'    => $o->getNom(),
            ':prenom' => $o->getPrenom(),
            ':ville'  => $o->getVille(),
            ':sexe'   => $o->getSexe()
        ]);
    }

    public function findAllApi() {
        $req = $this->connexion->getConnexion()->query("SELECT * FROM Etudiant");
        return $req->fetchAll(PDO::FETCH_ASSOC);
    }

    public function delete($o)    {}
    public function update($o)    {}
    public function findAll()     {}
    public function findById($id) {}
}
?>
```

---

### 🌐 Routes REST

**`ws/createEtudiant.php`** — Reçoit un POST, insère l'étudiant, retourne le tableau complet

```php
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    include_once '../service/EtudiantService.php';
    extract($_POST);
    $service = new EtudiantService();
    $service->create(new Etudiant(null, $nom, $prenom, $ville, $sexe));
    header('Content-Type: application/json');
    echo json_encode($service->findAllApi());
}
?>
```

**`ws/loadEtudiant.php`** — Retourne tous les étudiants en JSON (GET)

```php
<?php
include_once '../service/EtudiantService.php';
$service = new EtudiantService();
header('Content-Type: application/json');
echo json_encode($service->findAllApi());
?>
```

---

### 🧪 Tests avec Advanced REST Client / Postman

**Tester l'ajout :**

```
Méthode  : POST
URL      : http://localhost/projet/ws/createEtudiant.php
Body     : x-www-form-urlencoded
  nom    → Dupont
  prenom → Sara
  ville  → Casablanca
  sexe   → femme
```

**Tester la lecture :**

```
Méthode : GET
URL     : http://localhost/projet/ws/loadEtudiant.php
```

**Réponse attendue :**

```json
[
  {"id":"1","nom":"Lachgar","prenom":"Mohamed","ville":"Rabat","sexe":"homme"},
  {"id":"2","nom":"Dupont","prenom":"Sara","ville":"Casablanca","sexe":"femme"}
]
```

> 💡 Ces mêmes URLs sont ensuite réutilisées dans l'application Android, en remplaçant `localhost` par `10.0.2.2`.

---

## 📱 Partie 3 — Application Android

### Configuration initiale

**1.** Créer un projet Android Studio : nom `projetws`, type *Empty Activity*, langage **Java**, minSDK **26**.

**2.** Ajouter la permission Internet dans `AndroidManifest.xml` :

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

**3.** Déclarer les bibliothèques dans `build.gradle (Module: app)` :

```gradle
dependencies {
    implementation 'com.android.volley:volley:1.2.1'
    implementation 'com.google.code.gson:gson:2.10.1'
}
```

Puis cliquer sur **Sync Now**.

---

### 🔐 Configuration réseau (Android 9+)

Créer `res/xml/network_security_config.xml` :

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">10.0.2.2</domain>
    </domain-config>
</network-security-config>
```

Référencer dans `AndroidManifest.xml` :

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="true">
</application>
```

> 🌸 `10.0.2.2` est l'adresse qui désigne `localhost` depuis l'émulateur Android.

---

### 🎨 Styles UI `res/values/styles.xml`

```xml
<style name="text">
    <item name="android:textSize">17sp</item>
    <item name="android:paddingBottom">10dp</item>
    <item name="android:textColor">@android:color/holo_green_dark</item>
</style>

<style name="input">
    <item name="android:paddingBottom">20dp</item>
</style>
```

---

### 📄 Layout `activity_add_etudiant.xml`

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView android:text="Nom :"    style="@style/text" />
    <EditText android:id="@+id/nom"   style="@style/input"
        android:inputType="textPersonName" />

    <TextView android:text="Prénom :" style="@style/text" />
    <EditText android:id="@+id/prenom" style="@style/input"
        android:inputType="textPersonName" />

    <TextView android:text="Ville :"  style="@style/text" />
    <Spinner  android:id="@+id/ville"
        android:entries="@array/villes" style="@style/input" />

    <TextView android:text="Sexe :"   style="@style/text" />
    <RadioGroup android:orientation="horizontal">
        <RadioButton android:id="@+id/m" android:text="Homme" />
        <RadioButton android:id="@+id/f" android:text="Femme" />
    </RadioGroup>

    <Button android:id="@+id/add"
        android:text="Ajouter"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

---

### ☕ `beans/Etudiant.java`

```java
package com.example.projetws.beans;

public class Etudiant {
    private int id;
    private String nom, prenom, ville, sexe;

    public Etudiant() {}

    public Etudiant(int id, String nom, String prenom, String ville, String sexe) {
        this.id     = id;
        this.nom    = nom;
        this.prenom = prenom;
        this.ville  = ville;
        this.sexe   = sexe;
    }

    @Override
    public String toString() {
        return "Etudiant{"
            + "id="      + id
            + ", nom='"  + nom    + '\''
            + ", prenom='"+ prenom + '\''
            + ", ville='"  + ville   + '\''
            + ", sexe='"   + sexe    + '\''
            + '}';
    }
}
```

---

### ☕ `AddEtudiant.java` — Activité principale

```java
public class AddEtudiant extends AppCompatActivity implements View.OnClickListener {

    private EditText nom, prenom;
    private Spinner  ville;
    private RadioButton m, f;
    private Button   add;
    private RequestQueue file;

    private static final String ENDPOINT = "http://10.0.2.2/projet/ws/createEtudiant.php";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_etudiant);

        nom    = findViewById(R.id.nom);
        prenom = findViewById(R.id.prenom);
        ville  = findViewById(R.id.ville);
        m      = findViewById(R.id.m);
        f      = findViewById(R.id.f);
        add    = findViewById(R.id.add);

        file = Volley.newRequestQueue(this);
        add.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        if (v == add) soumettre();
    }

    private void soumettre() {
        StringRequest req = new StringRequest(Request.Method.POST, ENDPOINT,
            reponse -> {
                Type t = new TypeToken<Collection<Etudiant>>(){}.getType();
                Collection<Etudiant> liste = new Gson().fromJson(reponse, t);
                for (Etudiant e : liste) Log.d("ETUDIANT", e.toString());
            },
            erreur -> Log.e("VOLLEY", erreur.getMessage())) {

            @Override
            protected Map<String, String> getParams() {
                Map<String, String> champs = new HashMap<>();
                champs.put("nom",    nom.getText().toString());
                champs.put("prenom", prenom.getText().toString());
                champs.put("ville",  ville.getSelectedItem().toString());
                champs.put("sexe",   m.isChecked() ? "homme" : "femme");
                return champs;
            }
        };
        file.add(req);
    }
}
```

---

### 📊 Résultat dans Logcat

```
D/ETUDIANT: Etudiant{id=1, nom='LACHGAR', prenom='Mohamed', ville='Rabat', sexe='homme'}
D/ETUDIANT: Etudiant{id=2, nom='SAFI', prenom='Amine', ville='Marrakech', sexe='homme'}
```

---

## 🎬 Démonstration vidéo


https://github.com/user-attachments/assets/b39df2f6-5db9-443e-a9ad-44dbde53f7dc


> 📹 *Une vidéo de démonstration complète du laboratoire est disponible — elle couvre l'installation de XAMPP, la création des services PHP, les tests via ARC, et l'intégration Android.*

---

## 🌷 Challenge — Pour aller plus loin

- [ ] Créer une activité **`ListeEtudiantsActivity`** affichant tous les enregistrements dans un `RecyclerView`
- [ ] Au clic sur un enregistrement, ouvrir un **dialogue de modification ou suppression**
- [ ] Ajouter une **boîte de confirmation** avant toute suppression définitive
- [ ] **Actualiser automatiquement** l'affichage après chaque opération CRUD

---

## 💻 Environnement technique

```
Serveur local  : XAMPP (Apache 2.4 + MySQL 8)
Backend        : PHP 8 avec PDO (architecture MVC légère)
Test API       : Advanced REST Client (extension Chrome)
Mobile         : Android Studio · Java · minSDK 26
Réseau         : Volley 1.2.1
Sérialisation  : Gson 2.10.1
```

---

<div align="center">

*🌸 TP réalisé dans le cadre du module Développement Mobile 🌸*

</div>
