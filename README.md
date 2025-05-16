# TP1 - Injection de dépendances en Java

**Nom :** Sehait 
**Prénom :** Mayssam
**Module :** Systèmes Distribués - Master SDIA   


Ce projet a été réalisé dans le cadre du TP1 du module Systèmes Distribués.  
L’objectif est d’illustrer les différentes méthodes d’injection de dépendances en Java,  
avec et sans le framework Spring, en utilisant notamment les annotations et la configuration XML.


## Structure du projet

L'application est divisée en plusieurs packages :
- `net.mayssam.dao` : Contient les classes d'accès aux données.
- `net.mayssam.ext` : Contient une autre implémentation de DAO.
- `net.mayssam.metier` : Contient la logique métier.
- `net.mayssam.pres` : Contient les classes de présentation (différentes manières d’initialiser l’application).



## Interfaces et Implémentations

### `IDao`

Interface définissant la méthode `getData()` qui retourne une température.

```java
public interface IDao {
    double getData();
}
```

### `DaoImpl`

Implémentation de `IDao` utilisant une base de données (ou simulée ici par un simple retour de valeur).

```java
@Repository("d")
public class DaoImpl implements IDao {
    public double getData() {
        System.out.println("Version base de données");
        return 34;
    }
}
```

### `DaoImplV2`

Deuxième version du DAO, simulant des capteurs.

```java
@Repository("d2")
public class DaoImplV2 implements IDao {
    public double getData() {
        System.out.println("Version capteurs ....");
        return 12;
    }
}
```

### `IMetier`

Interface métier définissant une méthode `calcul()`.

```java
public interface IMetier {
    double calcul();
}
```

### `MetierImpl`

Classe métier qui utilise un objet `IDao` pour faire un calcul complexe à partir des données.

```java
@Service("metier")
public class MetierImpl implements IMetier {
    private IDao dao;

    public MetierImpl(@Qualifier("d2") IDao dao) {
        this.dao = dao;
    }

    public double calcul() {
        double t = dao.getData();
        return t * 12 * Math.PI / 2 * Math.cos(t);
    }

    public void setDao(IDao dao) {
        this.dao = dao;
    }
}
```



## Méthodes d’Injection utilisées

### 1. **Injection via le constructeur** (dans `Pres1` et `Pres2`)

```java
DaoImplV2 d = new DaoImplV2();
MetierImpl metier = new MetierImpl(d);
System.out.println("RES= " + metier.calcul());
```

### 2. **Injection via setter** (commentée dans `Pres1`, utilisable en remplaçant la ligne constructeur par `setDao()`)

```java
metier.setDao(d);
```

### 3. **Injection dynamique via réflexion** (dans `Pres2` avec un fichier `config.txt`)

```java
String daoClassName = scanner.nextLine(); // DaoImpl ou DaoImplV2
Class cDao = Class.forName(daoClassName);
IDao d = (IDao) cDao.newInstance();

String metierClassName = scanner.nextLine();
IMetier metier = (IMetier) cMetier.getConstructor(IDao.class).newInstance(d);
```

Fichier `config.txt` :

```
net.mayssam.dao.DaoImpl
net.mayssam.metier.MetierImpl
```

### 4. **Injection avec Spring via XML** (`PresSpringXML`)

```xml
<!-- config.xml -->
<bean id="dao" class="net.mayssam.dao.DaoImpl"/>
<bean id="metier" class="net.mayssam.metier.MetierImpl">
    <constructor-arg ref="dao"/>
</bean>
```

```java
ApplicationContext context = new ClassPathXmlApplicationContext("config.xml");
IMetier metier = context.getBean(IMetier.class);
```

### 5. **Injection avec Spring via annotations** (`PresSpringAnnotation`)

Utilisation des annotations `@Component`, `@Service`, `@Repository`, `@Autowired` et `@Qualifier` pour laisser Spring injecter automatiquement les beans.

```java
ApplicationContext context = new AnnotationConfigApplicationContext("net.mayssam");
IMetier metier = context.getBean(IMetier.class);
System.out.println("RES=" + metier.calcul());
```


## Résultats des exécutions

Chaque mode d’injection imprime un résultat du calcul accompagné d’un message indiquant la source des données (`Version base de données` ou `Version capteurs ....`).

### Injection par constructeur
![Pres1](images/pres1.png)

### Injection dynamique via Réflexion
![Pres2](images/pres2.png)

### Spring – Injection avec Annotations
![Spring Annotation](images/presSpringAn.png)

### Spring – Injection avec XML
![Spring XML](images/presSpringXML.png)




## Résumé 

- Mise en place d'une architecture en couches avec DAO et service métier.
- Injection de dépendances en Java sans framework (via constructeur, setter, réflexion).
- Utilisation de Spring pour gérer les dépendances :
  - via fichier XML (`ApplicationContext` avec `ClassPathXmlApplicationContext`)
  - via annotations (`AnnotationConfigApplicationContext`)
- Utilisation de `@Component`, `@Repository`, `@Service`, `@Autowired`, `@Qualifier`.

---

## Liens utiles

- [Dépôt GitHub du projet TP1](https://github.com/MayssamSe/tp1)
- [Documentation officielle Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html)

