Permessi
========

Moodle prevede un meccanismo autorizzativo per gli utenti, si tratta di un meccanismo con alcune particolarita' rispetto a simili altri sistemi, ma per quanto possa risultare complesso permette di avere notevoli funzionalita'.

Vi sono una serie di concetti che concorrono nel meccanismo autorizzativo:


* **Ruoli**
* **Capacita**'
* **Contesti**

Contesti
========

Moodle prevede diversi livelli al quale si applicano le autorizzazioni, lo stesso utente puo' ad esempio essere un docente in un corso, ma essere uno studente in un altro, e potrebbe ad esempio anche avere accesso alla reposrtistica del sito.

Questi livelli vengono definiti **contesti**. Le autorizzazioni saranno possibili e/o avranno effetto a seconda del contesto in cui ci troviamo.

I contesti sono organizzati in una gerarchia e sono:

* `context_system` ovvero il contesto di sistema, ce ne e' solo uno ed ha le autorizzazioni generali del sito
* `context_user` il contesto dell'utente, ve ne e' uno per ogni utente.
* `context_coursecat` un contesto per categoria di corsi
* `context_course` un contesto per corso
* `context_module` un contesto per attivita' del corso
* `context_block` un contesto per ogni blocco

I contesti utente sono contenuti in quello di sistema.

I contesti di categorie sono anche loro contenuti in quello di sistema, a loro volta possono contenere altri contesti di categoria, o contesti di corsi. Questi ultimi contengono i contesti di attivita'.

I contesti di blocco possono essere contenuti in qualsiasi altro contesto.

All'interno del codice bisogna spesso recuperare il contesto in uso, per fare cio' esistono diversi metodi di aiuto:

```php
$systemcontext = context_system::instance();
$usercontext = context_user::instance($user->id);
$categorycontext = context_coursecat::instance($category->id);
$coursecontext = context_course::instance($course->id);
$contextmodule = context_module::instance($cm->id);
$contextblock = context_block::instance($this->instance->id);
```

ed e' possibile recuperare il contesto per id:

```php
$context = context::instance_by_id($contextid);
```

Il contesto viene ad esempio consumeto dai metodi per verificare le effettive autorizzazioni dell'utente. E' abbastanza pratico semplicemente assegnarlo ad una variabile nelle prime righe della pagina.



