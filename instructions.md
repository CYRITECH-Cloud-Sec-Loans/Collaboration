Pour créer un hook dans Dolibarr qui s'active lorsqu'un utilisateur soumet un formulaire à l'URL spécifiée, comme https://www.cyritech.com/corporate/public/partnership/new.php, vous devrez suivre ces étapes :

Créer un module personnalisé : Si ce n'est pas déjà fait, créez un module personnalisé dans Dolibarr. Cela inclut la création d'une structure de répertoires pour le module et l'ajout d'un fichier mymodule.class.php pour définir le hook.

Définir le hook : Créez un fichier PHP dans le répertoire de votre module pour gérer le hook.

Configurer le formulaire externe : Vous devrez faire en sorte que le formulaire sur https://www.cyritech.com/corporate/public/partnership/new.php envoie les données à Dolibarr.

Voici comment procéder pour configurer un hook personnalisé pour gérer une demande de partenariat et envoyer un e-mail :

1. Créez le Module Personnalisé
Créez un répertoire pour votre module sous htdocs/custom/. Par exemple, nommons le module partnership.

bash
Copy code
htdocs/custom/partnership/
Dans ce répertoire, créez les fichiers nécessaires :

partnership.module.php : pour définir le module.
core/hooks/partnership.class.php : pour le code du hook.
2. Définissez le Module
Dans partnership.module.php :

php
Copy code
<?php

require '../core/lib/admin.lib.php';

$langs->load("admin");

class Partnership extends DolibarrModules
{
    public function __construct($db)
    {
        $this->db = $db;
        $this->numero = 104; // Numéro du module
        $this->rights_class = 'partnership';
        $this->family = 'custom';
        $this->name = 'Partnership';
        $this->description = 'Gestion des demandes de partenariat';
        $this->version = '1.0';
        $this->const_name = 'MAIN_MODULE_PARTNERSHIP';
        $this->picto = 'partnership';
        
        // Définir le hook
        $this->module_parts = array(
            'hooks' => array('partnership'),
        );
    }
}
?>
3. Créez le Fichier du Hook
Dans core/hooks/partnership.class.php :

php
Copy code
<?php

include_once '../core/lib/admin.lib.php';

class PartnershipHooks
{
    public function __construct($db, $langs, $conf)
    {
        // Initialisation si nécessaire
    }

    public function hookActionPartnership(&$parameters, &$object, &$action, $hookmanager)
    {
        // Vérifier si l'action est liée au formulaire de partenariat
        if ($parameters['context'] == 'formSubmit' && isset($parameters['formtype']) && $parameters['formtype'] == 'partnership') {
            $this->createPartnershipTicket($parameters);
            $this->sendEmail($parameters);
        }
    }

    private function createPartnershipTicket($parameters)
    {
        // Exemple de création d'un ticket de partenariat
        $ticket = new Ticket($this->db);
        $ticket->socid = $parameters['socid']; // ID de la société
        $ticket->title = "Nouvelle demande de partenariat";
        $ticket->message = "Demande de partenariat reçue de : " . $parameters['name'];
        $ticket->create($parameters['user']);
    }

    private function sendEmail($parameters)
    {
        global $conf, $langs;

        $subject = "Nouvelle demande de partenariat";
        $message = "Bonjour,\n\nNous avons reçu une nouvelle demande de partenariat.\n\nDétails de la demande :\n\nNom : " . $parameters['name'] . "\nEmail : " . $parameters['email'] . "\nMessage : " . $parameters['message'] . "\n\nMerci.";

        $emailto = $conf->global->MAIN_MAIL_SENDER; // Adresse e-mail de l'expéditeur ou celle de l'administrateur

        $mail = new TMail();
        $mail->set_subject($subject);
        $mail->set_message($message);
        $mail->set_to($emailto);
        $mail->send();
    }
}
?>
4. Configurer le Formulaire Externe
Vous devez vous assurer que le formulaire externe envoie les données à Dolibarr, généralement via un appel HTTP POST. Cela peut nécessiter des ajustements côté formulaire pour appeler un script PHP sur votre serveur Dolibarr qui traitera les données et déclenchera le hook.

5. Activer le Module
Assurez-vous que votre module est activé dans l'interface d'administration de Dolibarr.

Remarque : Le code ci-dessus est un exemple simplifié. Vous devrez ajuster les parties concernant la création du ticket et l'envoi des e-mails selon la configuration de votre Dolibarr et vos besoins spécifiques. Assurez-vous de tester le module dans un environnement de développement avant de le mettre en production.





You said:
Documenter intégreration ds dolibar étape par étape 
ChatGPT said:
ChatGPT
Pour intégrer un hook personnalisé dans Dolibarr afin de traiter une demande de partenariat depuis un formulaire externe, suivez les étapes ci-dessous :

1. Préparer l'Environnement
Installer Dolibarr : Assurez-vous que Dolibarr est correctement installé et fonctionnel.
Créer un Module Personnalisé : Vous devez avoir des droits d’administrateur sur Dolibarr pour créer des modules personnalisés.
2. Créer un Module Personnalisé
2.1 Créer la Structure du Module
Créer le Répertoire du Module : Dans le répertoire htdocs/custom/, créez un dossier pour votre module. Par exemple, partnership.

sh
Copy code
mkdir htdocs/custom/partnership
Créer les Fichiers de Module : Créez les fichiers de base dans ce répertoire. Voici les fichiers essentiels :

partnership.module.php : Déclaration du module.
core/hooks/partnership.class.php : Code pour le hook.
2.2 Définir le Module
Dans partnership.module.php :

php
Copy code
<?php

require '../core/lib/admin.lib.php';

$langs->load("admin");

class Partnership extends DolibarrModules
{
    public function __construct($db)
    {
        $this->db = $db;
        $this->numero = 104; // Numéro du module
        $this->rights_class = 'partnership';
        $this->family = 'custom';
        $this->name = 'Partnership';
        $this->description = 'Gestion des demandes de partenariat';
        $this->version = '1.0';
        $this->const_name = 'MAIN_MODULE_PARTNERSHIP';
        $this->picto = 'partnership';
        
        $this->module_parts = array(
            'hooks' => array('partnership'),
        );
    }
}
?>
3. Créer le Hook
3.1 Code du Hook
Dans core/hooks/partnership.class.php :

php
Copy code
<?php

include_once '../core/lib/admin.lib.php';

class PartnershipHooks
{
    public function __construct($db, $langs, $conf)
    {
        // Initialisation si nécessaire
    }

    public function hookActionPartnership(&$parameters, &$object, &$action, $hookmanager)
    {
        if ($parameters['context'] == 'formSubmit' && isset($parameters['formtype']) && $parameters['formtype'] == 'partnership') {
            $this->createPartnershipTicket($parameters);
            $this->sendEmail($parameters);
        }
    }

    private function createPartnershipTicket($parameters)
    {
        $ticket = new Ticket($this->db);
        $ticket->socid = $parameters['socid']; // ID de la société
        $ticket->title = "Nouvelle demande de partenariat";
        $ticket->message = "Demande de partenariat reçue de : " . $parameters['name'];
        $ticket->create($parameters['user']);
    }

    private function sendEmail($parameters)
    {
        global $conf, $langs;

        $subject = "Nouvelle demande de partenariat";
        $message = "Bonjour,\n\nNous avons reçu une nouvelle demande de partenariat.\n\nDétails de la demande :\n\nNom : " . $parameters['name'] . "\nEmail : " . $parameters['email'] . "\nMessage : " . $parameters['message'] . "\n\nMerci.";

        $emailto = $conf->global->MAIN_MAIL_SENDER;

        $mail = new TMail();
        $mail->set_subject($subject);
        $mail->set_message($message);
        $mail->set_to($emailto);
        $mail->send();
    }
}
?>
4. Configurer le Formulaire Externe
Envoyer les Données au Serveur Dolibarr : Modifiez le formulaire sur https://www.cyritech.com/corporate/public/partnership/new.php pour qu’il envoie les données à un script PHP sur votre serveur Dolibarr.

Par exemple, envoyez les données via une requête HTTP POST à https://votre-dolibarr.com/custom/partnership/receive.php.
Créer le Script de Réception : Ce script traitera les données du formulaire et déclenchera le hook.

Exemple de receive.php :

php
Copy code
<?php

include '../../main.inc.php';
include '../../core/lib/admin.lib.php';
include '../class/partnership.class.php'; // Inclure votre module personnalisé

// Récupérer les données du formulaire
$name = $_POST['name'];
$email = $_POST['email'];
$message = $_POST['message'];

// Initialiser le module
$hookmanager = new HookManager($db);
$hookmanager->initHooks(array('partnership'));

$parameters = array(
    'context' => 'formSubmit',
    'formtype' => 'partnership',
    'name' => $name,
    'email' => $email,
    'message' => $message
);

$object = new stdClass();
$action = '';

$hookmanager->callHooks('actionPartnership', $parameters, $object, $action);

// Redirection ou confirmation
echo 'Demande reçue.';
?>
5. Activer le Module
Accédez à l’Interface d’Administration de Dolibarr :

Connectez-vous à Dolibarr.
Allez dans l’onglet "Modules/Applications".
Trouvez votre module "Partnership" et activez-le.
Vérifier le Fonctionnement :

Testez le formulaire externe pour vous assurer que les données sont correctement envoyées à Dolibarr.
Vérifiez que le ticket est créé et que l’e-mail est envoyé comme prévu.
6. Tester et Valider
Test de Fonctionnalité : Soumettez des formulaires de test pour vérifier que le hook fonctionne correctement.
Dépannage : En cas de problème, vérifiez les journaux d'erreurs PHP et Dolibarr pour diagnostiquer les problèmes.
Conclusion
Cette intégration permet à Dolibarr de traiter les demandes de partenariat soumises via un formulaire externe. Assurez-vous de personnaliser les chemins, variables et configurations pour correspondre à votre environnement spécifique.
