# wpfThreads
In questa prova di laboratorio abbiamo creato un pulsante che quando viene spinto fa partire due thread che contano fino a un numero prestabilito allo stesso tempo e successivamente abbiamo creato un secondo pulsante che elimina i due thread.
## Spiegazione prova di laboratorio
### Thread sleep
Thread.sleep permette di addormentare i thread in un determinato periodo di tempo
all’interno del thread della user interface non posso però impiccare il mio processo principale e non è mai buona pratica fare degli event end.
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	for(int x=0; x<GIRI ; x++)
	{
		lblCounterl.Text = x.ToString();
		Thread.Sleep(100);
	}

}
```
Bisogna lanciare un thread separato alla velocità che gli pare ma bisogna lasciare libero il thread della GUI.

Bisogna prima di tutto bisogna fare un nuovo metodo che non torna nulla così la GUI continua a girare indipendentemente da tutto.
```
private void incrementa()
{
	for(int x = 0;x < GIRI; x++)
	{
		lblCounterl.Text = x.ToString();
		Thread.Sleep(100);
	}
}
```
I due thread non possono utilizzare l’uno le risorse dell’altro senza pagare dazio perciò serve un lasciapassare perciò si utilizza l’oggetto dispatcher (zona critica) e dentro di esso si fanno cose che solitamente non potrebbero essere fatte.
```
private void incrementa()
{
	for(int x = 0;x < GIRI; x++)
	{
		Dispatcher.Invoke(
			() =>
			{
				lblCounterl.Text = x.ToString();
			}
		);

		Thread.Sleep(100);
	}
}
```
Dopo aver invocato il dispatcher nella interfaccia grafica verrà visualizzato il conteggio in tempo reale mentre prima dava solo il risultato finale però il conteggio sfarfalla perchè due thread stanno lavorando sulla stessa variabile e posso far lavorare anche più di due thread alla volta.
i thread possono avere più stati, eccoli elencati : 

![thread sleep 5](https://user-images.githubusercontent.com/116788504/231789283-76a885a4-6b65-4557-8901-870a88866a20.jpg)

Se invece metto due contatori essi non andranno mai d’accordo e si fermeranno a numeri diversi perché c’è concorrenza nella variabile e finchè non si è finito di prendere il risultato e mostrarlo nessuno deve disturbare l’operazione e quindi per risolvere questo problema si usa lock:
### Lock

![thread sleep 6](https://user-images.githubusercontent.com/116788504/231789474-86f37db9-ce83-41fd-8207-7f4604283a3f.jpg)

```
private void incrementa1()
{
	for(int x = 0;x < GIRI; x++)
	{
		lock(_locker)
		{
			_counter++;
		}
		Dispatcher.Invoke(
			() =>
			{
				lblCounterl.Text = _counter.ToString();
			}
		);

		Thread.Sleep(1);
	}
}
```
lock per funzionare ha bisogno di un oggetto:
```
static readonly object _locker = new object();
```
ora funziona tutto quasi alla perfezione:

![thread sleep 9](https://user-images.githubusercontent.com/116788504/231789825-a62722db-ea2e-46e5-b50f-6a6babd6c026.jpg)

Il problema è che nessuno aggiorna la label di incrementa1 per questo un contatore conta male e uno bene.

![wpf string1](https://user-images.githubusercontent.com/116788504/231790075-4ce4062a-4663-4e16-970b-af5ffeda9bfb.jpg)

Per questo problema serve un lock che serve per far si che il processo non sia bloccato e quindi funge da semaforo.
Il semaforo è un intero che non può essere negativo.
Con la signal prendo il contatore e lo decrementa di uno ogni volta fino ad arrivare a 0.
Wait invece è una procedura bloccante che si sblocca quando i processi sono finiti.
Il semaforo si chiama countdwonevent
```
CowntdownEvent semaforo = new CountdownEvent(2);
```
e lo inizializziamo a 2. 
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	Thread thread1 = new Thread(incrementa1);
	thread1.Start();

	Thread thread2 = new Thread(incrementa2);
	thread2.Start();

	semaforo = new CountdownEvent(2);
}
```
Il semaforo.wait lavora fin quando i signal sono finiti e lo segnalano al wait.

![wpf string4](https://user-images.githubusercontent.com/116788504/231790538-3062812a-bd2a-4333-8292-ed2d19958ac5.jpg)

Per far si che capiamo che tutto è andato a buon fine facciamo stampare una stringa
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	Thread thread1 = new Thread(incrementa1);
	thread1.Start();

	Thread thread2 = new Thread(incrementa2);
	thread2.Start();

	semaforo = new CountdownEvent(2);
	semaforo.Wait();

	MessageBox.Show("Fine!!");
}
```
Però c'è un errore, i semafori non possono essere messi dentro un event end.
perciò dobbiamo lanciare il terzo thread e abbiamo bisogno di un metodo.
Siccome il metodo uno e due non tornano niente la strada migliore è creare un altro metodo dove infilarci il wait .
### Wait
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	Thread thread1 = new Thread(incrementa1);
	thread1.Start();

	Thread thread2 = new Thread(incrementa2);
	thread2.Start();

	semaforo = new CountdownEvent(2);
	
	Thread thread3 = new Thread(incrementa3);
	thread3.Start();

}

private void attendi()
{
	semaforo.Wait();
}
```
Devo fare un altro dispatcher in questo metodo perché ha bisogno anche lui di bucare il bucabile.
### Dispatcher
```
private void attendi()
{
	semaforo.Wait();
	Dispatcher.Invoke(
		() =>
		
		{
			lblCounterl.Text = _counter.ToString();
			lblCounter2.Text = _counter.ToString();
		}
        );

}
```
Per far capire che tutto è finito facciamo un messaggio di avviso.
```
private void attendi()
{
	semaforo.Wait();
	Dispatcher.Invoke(
		() =>

		{
			MessageBox.Show("Finito!!");
			lblCounterl.Text = _counter.ToString();
			lblCounter2.Text = _counter.ToString();
		}
      );

}
```
### Invoke

Il dispatcher invoke ha un parametro di tipo lambda perchè tutto quello che sta all’interno delle parentesi graffe sono codice.
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	Thread thread1 = new Thread(incrementa1);
	thread1.Start();

	Thread thread2 = new Thread(incrementa2);
	thread2.Start();

	semaforo = new CountdownEvent(2);
	
	Thread thread3 = new Thread(

	    () =>

	    {
		 semaforo.Wait();
		 Dispatcher.Invoke(

		     () =>

		     {
				MessageBox.Show("Finito!!");
			      lblCounterl.Text = _counter.ToString();
			      lblCounter2.Text = _counter.ToString();
		     }
	    }
      );
      Thread3.Start();
}

```
Per comodità e leggibilità facciamo tutto come nella foto precedente.
Se clicco una seconda volta il button il semaforo si spacca perchè vengono a meno i reference del primo semaforo.
```
private void incrementa2()
        {
            for (int x = 0; x < GIRI2; x++)
            {
                lock (_locker)
                {
                    _counter2++;
                }

                Dispatcher.Invoke(

                    () =>

                    {
                        lblCounter2.Text = _counter2.ToString();
                       
                    }

                );

                Thread.Sleep(2);
            }
            semaforo.Signal();
        }
```
Per questo diamo il nome al pulsante e lo disabilitiamo dopo averlo attivato per la prima volta.
Ora mettiamo il bottone a true e in questo modo lo possiamo attivare quando vogliamo e sarà sempre coordinato.
### Fine
```
private void Button_Click(object sender, RoutedEventArgs e)
{
	btnGo.IsEnabled = false;
	Thread thread1 = new Thread(incrementa1);
	thread1.Start();

	Thread thread2 = new Thread(incrementa2);
	thread2.Start();

	semaforo = new CountdownEvent(2);
	
	Thread thread3 = new Thread(

	    () =>

	    {
		 semaforo.Wait();
		 Dispatcher.Invoke(

		     () =>

		     {
				MessageBox.Show("Finito!!");
			        lblCounterl.Text = _counter.ToString();
			        lblCounter2.Text = _counter.ToString();
				btnGo.IsEnabled = true;
		     }
	    }
      );
      Thread3.Start();
}
```


