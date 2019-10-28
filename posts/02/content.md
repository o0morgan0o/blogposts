# Mais c'est quoi un Arpégiatteur ?

C'est un outil pour tout fan de musique électronique, cela permet de réaliser les notes d'un accord de manière répétitive et dans un ordre précis (ou non si c'est l'effet voulu). Ce fut un projet que j'ai réalisé l'année dernière et j'en profites ici pour résumer et détailler les différentes étapes de la réalisation.

# Les fonctions que je voulais développer

Je voulais un équipement capable de :
1. Générer un arpège d'accord en appuyant sur une des notes.
2. Avoir un feedback visuel des notes jouées.
3. Pouvoir régler l'octave utilisée, le type d'arpège joué, et la durée des notes.
4. Pouvoir régler le bpm et le nombre de notes jouées par arpège.
5. Communiquer les notes en MIDI avec un synthétiseur ou un ordinateur.

Au final cela représentait un sacré challenge, la plupart des fonctions étant en fait pas si évidentes à implémenter.

# Déroulement du projet

Ayant fait ce projet l'année dernière le déroulement de l'ensemble du projet n'est plus complétement frais dans mon esprit mais je vais essayer de refaire en gros la démarche.
Le code et le gros du déroulement est disponible publiquement sur mon [repo Github](https://github.com/o0morgan0o/HardwareArpegiator).

**Tout d'abord le matos:**

La plupart des composants sont standards et pour ma part je les avais achetés sur le site [Semageek](https://https://boutique.semageek.com/fr/).
Ce qui rends l'ensemble assez cool est pour moi les bouttons type arcade lumineux qui rappellent les meilleurs heures de ma période Arcade de gamer !

1. Arduino MEGA 2560 (x1)
2. Bouttons Arcade lumineux (x12)
3. Bouttons Arcade standard (x3)
4. Push buttons.(x6)
5. Boutton biposition (x1)
6. LEDs (x9);
7. Potentiometres 3 pins (x2)
8. Ecran cristaux liquides (x1)
9. Prise MIDI 5pins (x1)
10. Breadboards ou plaques de prototypage.
11. Des cables, des résistances.
12. Du café !

Ensuite il faut réussir à assembler tout ca.
Avec le recul je ferais les choses différemment pour optimiser l'ensemble et faire du **multiplexage** sur les entrées et sorties pour avoir quelque chose de plus clair et simple. 
Mais bon cela était dans mes premières expériences et m'a permis de me rendre compte de pas mal d'erreurs.

Voici un schéma simplifié de l'ensemble.

![alt text](https://camo.githubusercontent.com/28012efd6f88e5b9e009d4c36467aebd7b320c6d/68747470733a2f2f692e696d6775722e636f6d2f6c654332766e442e706e67 "schema de branchement")

J'envisage de reconstruire tout cela dans une sorte de v2 qui sera beaucoup plus optimisée. 

Après toute la partie soudure et assemblage (et surtout code!) nous arrivons à un petit résultat tel que celui ci (*fait avec des matériaux de récup comme vous pouvez voir!!*):

![alt text](https://www.morgan-thibert.com/src/img/blog/02/Arp01.jpg "premier draft")


Et le hasard fait bien les choses puisqu'à l'époque je venais de faire l'acquisition d'une imprimante 3d ! On en reparleras très certainement car j'ai beaucoup de petits projets qui prennent forme grâce à l'imprimante.
Donc du coup nous voici maintenant avec une petite coque réalisé moi même après avoir appris les bases de **Fusion360** (outil de CAO).

<img src="https://camo.githubusercontent.com/03819b1892a8c078c99e6bb91f33ac644ad49dc7/68747470733a2f2f692e696d6775722e636f6d2f6a36795843515a2e706e67" />

(Le fichier stl est d'ailleurs disponible dans le repo github).


## Le code !

J'aimerais en profiter ici pour expliquer quelques difficultés sur le code arduino du fichier.
L'un des éléments qui m'a donné du fil à retorder est la gestion des NoteOn et NoteOff afin d'avoir des notes qui ont bien la bonne durée.

Exemple
```

void changeArpeggio(int midiNote)

{
  switch (arpDirection)
  {
  case 0: //cas pour arpege ascendant
    temporaryDisplay(String(arpDirection), "up");
    recalcNotesOffset(arpeggioLength);
    for (int i = 0; i < arpeggioLength; i++)
    {
      arpeggio[i] = midiNote + arpeggioNotesOffset[i];
    }
    myMIDInote = midiNote;
    break;
  case 1: //cas pour arpege descendant
    temporaryDisplay(String(arpDirection), "down");
    for (int i = 0; i < arpeggioLength; i++)
    {
      arpeggio[arpeggioLength - 1 - i] = midiNote + arpeggioNotesOffset[i];
    }
    myMIDInote = midiNote;
    break;
  case 2: //cas pour arpege aleatoire
    temporaryDisplay(String(arpDirection), "up and down");
    if (arpeggioLength % 2 == 0)
    {
      for (int i = 0; i < (arpeggioLength / 2 + 1); i++)
      {
        int bufferValue = arpeggio[i];
        arpeggio[i] = midiNote + arpeggioNotesOffset[i];
        arpeggio[arpeggioLength - 1 - i] = bufferValue;
      }
      myMIDInote = midiNote;
    }
    else
    {
      for (int i = 0; i < (arpeggioLength / 2); i++)
      {
        arpeggio[i] = midiNote + arpeggioNotesOffset[i];
        arpeggio[arpeggioLength - 1 - i] = arpeggio[i];
      }
      myMIDInote = midiNote;
    }

    break;
  }
}
```
A chaque fois que l'on reconstruit un arpège (lorsqu'on appuie sur une note), il faut à la fois faire attention à bien "couper" les notes en cours au bon moment, mais également reconstruire deux arrays. Une pour les futurs NoteOn qui doivent être transmis, et une pour les NoteOff qui devront être envoyés pour stopper les notes. L'array devra aussi être mise à jour à chaque fois qu'il y aura un changement dans l'arpège, ou lorsque le BPM ou la durée des notes est mise à jour par l'utilisateur.

De manière générale la gestion des timings, et les inputs en temps réel et assez délicate en Arduino puisque l'Arduino fonctionne plutôt par loop. Il faut donc gérer les évènement qui peuvent arriver n'importe quand d'une manière un peu différente...


## Au final :

Finallement je suis assez content du résultat pour un premier projet type Hardware aussi "compliqué" par rapport à mon expérience dans ce domaine. Ca m'a permit de comprendre beaucoup mieux par exemple la transmission de signaux **MIDI** à un appareil externe. (Rappellons que le [MIDI](https://fr.wikipedia.org/wiki/Musical_Instrument_Digital_Interface) est une norme qui date de bientôt 40 ans et qui est toujours aussi fiable, c'est assez remarquable).

Voici le resultat vidéo en reliant mon arpégiatteur à un synthétiseur sous Ableton.

[![IMAGE ALT TEXT](https://img.youtube.com/vi/ljZOk-fAyPg/0.jpg)](http://www.youtube.com/watch?v=ljZOk-fAyPg "Video Title")

***Ca marche !!***

Et si on soulève le capot:

![alt text](https://www.morgan-thibert.com/src/img/blog/02/Arp02.jpg "premier draft")

# Version 2

J'espère cette année pouvoir prendre le temps de refactoriser mon code et de reconstruire l'ensemble de manière plus ordonnée. J'ai une sorte de proto qui fonctionne, mais redessiner l'ensemble pour avoir quelque chose de vraiment fonctionnel et robuste dans une utilisation quotidienne me tente bien !

Affaire à suivre !