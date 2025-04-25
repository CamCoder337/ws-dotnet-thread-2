### P10_Corbeille_[A4] PR10 - DEV_MSCS_WS2_Exar_V1-0 [EN-100]

## Q2 - Phylosophy of the code

The main presented code shows diverse of parallel programming in C#. Its
want to highlight :
-   Synced thread execution management
-   Usage of delegates and events for thread to thread communication 
  - Implementation of different parallelization methods (Parallel For, Parallel for each...)
- Synced access of shared resources using locked mechanism

## Q3 - Translate this code line by line

```csharp
using System;                    // Import des espaces de noms standard
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace test
{
    class Program
    {
        delegate void DELG();    // Déclaration d'un délégué sans paramètre ni retour
        delegate void EVT(object o);  // Déclaration d'un délégué avec un paramètre objet
        
        static event EVT evt;    // Déclaration d'un événement basé sur le délégué EVT
        static int loop;         // Variable partagée pour compter les terminaisons
        static object LOCK;      // Objet de verrouillage pour la synchronisation
        
        static void Main(string[] args)
        {
            DELG dlG_time = new DELG(time);  // Création d'un délégué pointant vers la méthode time
            DELG dlg_multiCast;              // Déclaration d'un délégué multicast (sans initialisation)
            IAsyncResult asyncR;             // Pour suivre l'état d'une opération asynchrone
            
            LOCK = new object();             // Initialisation de l'objet de verrouillage
            loop = 0;                        // Initialisation du compteur
            
            evt += new EVT(state_display);   // Abonnement à l'événement
            
            // Création d'un type anonyme avec deux propriétés message
            var an_type = new {msg_pext = "pextension", msg_noext="noextension"};
            
            // Création d'un thread qui appelle la méthode time
            System.Threading.Thread thd_timeInvok =
                new System.Threading.Thread(new System.Threading.ThreadStart(dlG_time.Invoke));
            
            // Création d'un thread qui utilise Parallel.For
            System.Threading.Thread thd_paraExt =
                new System.Threading.Thread(
                    new System.Threading.ThreadStart(() => {
                        // Boucle parallèle de 0 à 9
                        System.Threading.Parallel.For(0, 10, i => {
                            Console.WriteLine(@"{0}", an_type.msg_pext);
                            System.Threading.Thread.Sleep(1000);
                        });
                        // Déclenche l'événement quand terminé
                        evt((object)an_type.msg_pext);
                    }));
            
            // Création d'un thread qui utilise une boucle for classique
            System.Threading.Thread thd_noParaExt =
                new System.Threading.Thread(
                    new System.Threading.ThreadStart(()=> {
                        for (int i = 0; i < 10; i++) {
                            Console.WriteLine(@"{0}", an_type.msg_noext);
                            System.Threading.Thread.Sleep(1000);
                        };
                        // Déclenche l'événement quand terminé
                        evt((object)an_type.msg_noext);
                    }));
            
            // Invocation asynchrone du délégué time
            asyncR = dlG_time.BeginInvoke((async) => {
                // Callback à l'achèvement
                DELG d = (DELG)((System.Runtime.Remoting.Messaging.AsyncResult)async).AsyncDelegate;
                d.EndInvoke(async);
                Console.Write("Fin des traveaux");
            }, dlG_time);
            
            // Création d'un délégué multicast pour démarrer les deux threads
            dlg_multiCast = thd_paraExt.Start;
            dlg_multiCast += thd_noParaExt.Start;
            dlg_multiCast.Invoke();  // Démarrage des deux threads
            
            // Boucle d'attente jusqu'à la fin de l'opération asynchrone
            while (!asyncR.IsCompleted) { 
                Console.WriteLine("Travaux en cours..."); 
                System.Threading.Thread.Sleep(5000); 
            }
            
            Console.Read();  // Attente d'une saisie utilisateur
        }
        
        static void time()
        {
            long t = 0;
            lock (LOCK)  // Verrouillage pour l'accès concurrent
            {
                while (loop < 2)  // Continue jusqu'à ce que les deux autres threads terminent
                {
                    Console.WriteLine(t.ToString());
                    t += 1;
                    System.Threading.Thread.Sleep(1000);
                }
            }
        }
        
        static void state_display(object o)
        {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.WriteLine("{0} STOP", (string)o);
            Console.ForegroundColor = ConsoleColor.Gray;
            // Incrémente loop de manière thread-safe
            System.Threading.Interlocked.Increment(ref loop);
        }
    }
}
```
## Q4 - Write a simple program highlighting the use of Parallel.For

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        Parallel.For(0, 5, i =>
        {
            Console.WriteLine($"Task {i} started by thread {Task.CurrentId}");
            System.Threading.Thread.Sleep(1000);
        });
        Console.WriteLine("All tasks done.");
    }
}

```

## Q5 - Write a simple program highlighting the use of Parallel.ForEach

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var names = new List<string> { "Alpha", "Beta", "Gamma" };

        Parallel.ForEach(names, name =>
        {
            Console.WriteLine($"Processing {name} on thread {Task.CurrentId}");
            System.Threading.Thread.Sleep(500);
        });

        Console.WriteLine("Finished processing all names.");
    }
}

```


## Q6 -Write a simple program highlighting the use of Parallel.Invoke

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        Parallel.Invoke(
            () => Task1(),
            () => Task2(),
            () => Task3()
        );
    }

    static void Task1() => Console.WriteLine("Task 1 running");
    static void Task2() => Console.WriteLine("Task 2 running");
    static void Task3() => Console.WriteLine("Task 3 running");
}

```

## Q7 - Write a simple program highlighting the use of Task.Factory

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        Task[] tasks = new Task[3];

        for (int i = 0; i < tasks.Length; i++)
        {
            int taskNum = i; // évite les erreurs de fermeture
            tasks[i] = Task.Factory.StartNew(() =>
            {
                Console.WriteLine($"Task {taskNum} starting...");
                System.Threading.Thread.Sleep(1000);
                Console.WriteLine($"Task {taskNum} finished.");
            });
        }

        Task.WaitAll(tasks); // attend que toutes les tâches soient finies
        Console.WriteLine("All tasks completed.");
    }
}

```