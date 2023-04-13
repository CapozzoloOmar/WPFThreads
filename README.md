# wpfThreads
In questa prova di laboratorio abbiamo creato un pulsante che quando viene spinto fa partire due thread che contano fino a un numero prestabilito allo stesso tempo e successivamente abbiamo creato un secondo pulsante che elimina i due thread.
## Spiegazione prova di laboratorio
### Thread sleep
Thread.sleep permette di addormentare i thread in un determinato periodo di tempo
all’interno del thread della user interface non posso però impiccare il mio processo principale e non è mai buona pratica fare degli event end.
![thread sleep](https://user-images.githubusercontent.com/116788504/231788422-a001d22c-ae9f-4dcb-abcb-3679fdadea23.jpg)
Bisogna lanciare un thread separato alla velocità che gli pare ma bisogna lasciare libero il thread della GUI.
Bisogna prima di tutto bisogna fare un nuovo metodo che non torna nulla così la GUI continua a girare indipendentemente da tutto.

![thread sleep 3](https://user-images.githubusercontent.com/116788504/231788811-b728ac2a-ef6b-46ce-a909-b409049ee297.jpg)

I due thread non possono utilizzare l’uno le risorse dell’altro senza pagare dazio perciò serve un lasciapassare perciò si utilizza l’oggetto dispatcher (zona critica) e dentro di esso si fanno cose che solitamente non potrebbero essere fatte.
![thread sleep 4](https://user-images.githubusercontent.com/116788504/231789064-0e2dd572-d509-4cf3-bdd0-36c7d69ac2cd.jpg)

Dopo aver invocato il dispatcher nella interfaccia grafica verrà visualizzato il conteggio in tempo reale mentre prima dava solo il risultato finale però il conteggio sfarfalla perchè due thread stanno lavorando sulla stessa variabile e posso far lavorare anche più di due thread alla volta.
i thread possono avere più stati, eccoli elencati : 

![thread sleep 5](https://user-images.githubusercontent.com/116788504/231789283-76a885a4-6b65-4557-8901-870a88866a20.jpg)

Se invece metto due contatori essi non andranno mai d’accordo e si fermeranno a numeri diversi perché c’è concorrenza nella variabile e finchè non si è finito di prendere il risultato e mostrarlo nessuno deve disturbare l’operazione e quindi per risolvere questo problema si usa lock:
### Lock

![thread sleep 6](https://user-images.githubusercontent.com/116788504/231789474-86f37db9-ce83-41fd-8207-7f4604283a3f.jpg)

![thread sleep 7](https://user-images.githubusercontent.com/116788504/231789604-7fbf2962-08f0-480d-b804-0acddad41435.jpg)

lock per funzionare ha bisogno di un oggetto:

![thread sleep 8](https://user-images.githubusercontent.com/116788504/231789708-9d353838-4997-4552-b72e-f8a45c010d0e.jpg)

ora funziona tutto quasi alla perfezione:

![thread sleep 9](https://user-images.githubusercontent.com/116788504/231789825-a62722db-ea2e-46e5-b50f-6a6babd6c026.jpg)

Il problema è che nessuno aggiorna la label di incrementa1 per questo un contatore conta male e uno bene.
![wpf string1](https://user-images.githubusercontent.com/116788504/231790075-4ce4062a-4663-4e16-970b-af5ffeda9bfb.jpg)

Per questo problema serve un lock che serve per far si che il processo non sia bloccato e quindi funge da semaforo.
Il semaforo è un intero che non può essere negativo.
Con la signal prendo il contatore e lo decrementa di uno ogni volta fino ad arrivare a 0.
Wait invece è una procedura bloccante che si sblocca quando i processi sono finiti.
Il semaforo si chiama countdwonevent
![wpf string2](https://user-images.githubusercontent.com/116788504/231790220-5181e071-e983-4625-81ec-e470602f38b8.jpg)

e lo inizializziamo a 2. 

![wpf string3](https://user-images.githubusercontent.com/116788504/231790337-d137fdcd-f6a6-403b-8185-4eb2f4b84222.jpg)

Il semaforo.wait lavora fin quando i signal sono finiti e lo segnalano al wait.
![wpf string4](https://user-images.githubusercontent.com/116788504/231790538-3062812a-bd2a-4333-8292-ed2d19958ac5.jpg)

Per far si che capiamo che tutto è andato a buon fine facciamo stampare una stringa
![wpf string5](https://user-images.githubusercontent.com/116788504/231790670-fafc435f-e1d1-4742-8e87-f696bab3e749.jpg)

Però c'è un errore, i semafori non possono essere messi dentro un event end.
perciò dobbiamo lanciare il terzo thread e abbiamo bisogno di un metodo.
Siccome il metodo uno e due non tornano niente la strada migliore è creare un altro metodo dove infilarci il wait .
### Wait

![wpf string 6](https://user-images.githubusercontent.com/116788504/231790907-f5859209-02d2-4b2f-b602-52f391ac56e4.jpg)

Devo fare un altro dispatcher in questo metodo perché ha bisogno anche lui di bucare il bucabile.
### Dispatcher

![wpf string7](https://user-images.githubusercontent.com/116788504/231791018-f993ef4b-06c7-4a3a-be65-bae78b30d81a.jpg)

Per far capire che tutto è finito facciamo un messaggio di avviso.
![wpf string8](https://user-images.githubusercontent.com/116788504/231791170-2669800b-4107-4876-8e06-c66d5a4245ed.jpg)
### Invoke

Il dispatcher invoke ha un parametro di tipo lambda perchè tutto quello che sta all’interno delle parentesi graffe sono codice.
![wpf string9](https://user-images.githubusercontent.com/116788504/231791365-a015857a-9827-45f5-b774-a387f6dd1625.jpg)

Per comodità e leggibilità facciamo tutto come nella foto precedente.
Se clicco una seconda volta il button il semaforo si spacca perchè vengono a meno i reference del primo semaforo.
![wpf string10](https://user-images.githubusercontent.com/116788504/231791544-0b2acbcb-bb9c-4d93-ab0b-bf299e8e6406.jpg)

Per questo diamo il nome al pulsante e lo disabilitiamo dopo averlo attivato per la prima volta.
Ora mettiamo il bottone a true e in questo modo lo possiamo attivare quando vogliamo e sarà sempre coordinato.
### Fine

![wpf string12](https://user-images.githubusercontent.com/116788504/231791788-75ae6367-6215-469c-b995-b7b46a8d7068.jpg)


