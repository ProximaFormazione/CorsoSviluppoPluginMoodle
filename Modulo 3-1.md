Renderer
========

Finora abbiamo realizzato delle pagine utilizzando la variabile `$OUTPUT` base e lanciando direttamente comandi con la classe `html_writer` nel codice della pagina.

Per plugin semplici questo approccio e' assolutamente valido, ma puo' valere la pena avere una struttura diversa, dove la logica della grafica e' separata dalla logica di backend.

Vediamo ora una veloce carellata dei sistemi proposti da moodle.