# Adventuron games in French
How to develop a game in Adventuron in French (personal notes)


These are my personal notes are for Adventuron betabeta 1.0.0 v77, things might have changed so check the Adventuron documentation to know if these recommendations still apply.

## Language

Set the language to Spanish. Adventuron has certain support for Spanish that can be helpful also for French (e.g., accents support). Using Spanish it manages to distinguish quite well the relevant nouns in the sentence even if you the player use "avec" or other prepositions that do not exist in Spanish.

`language = spanish`

## Accents

Some players will not introduce the accents and you might still want it to work. Nothing needs to be done with á é í ó ú, Adventuron has deaccenting support for Spanish, just remember to use the noun with accent in the match commands. For the other symbols in French, there are two ways.
Add them in the vocabulary, this is useful when the verbs or nouns are used several times. Once you have done this, you can write only one of the aliases in your match commands.

```
vocabulary {
   : verb aliases = [brûler, bruler];      
   : noun aliases = [collègue, collegue];   
   : noun aliases = [baïonnette, baionnette];
   : noun aliases = [boîte, boite];
   : noun aliases = [cratère, cratere];
   : noun aliases = [forêt, foret];
   : noun aliases = [mèche, meche];   
}
```

Or you can just duplicate it in the match, this can be ok when the verbs or nouns are not used several times

```
: match "ouvrir boîte; ouvrir boite" {
}
```

## Basic commands

Some basic commands should be added as synonyms of existing Spanish ones.

```
vocabulary {
   : verb aliases = [s, sud];
   : verb aliases = [n, nord];
   : verb aliases = [e, est];
   : verb aliases = [o, ouest];
   : verb aliases = [x, examiner, inspecter];
   : verb aliases = [r, m, regarder];
   : verb aliases = [i, inventaire];
   : verb aliases = [salvar, sauver, sauvegarder];
   : verb aliases = [cargar, charger];
   : verb aliases = [p, prendre, saisir, agripper, arracher, attraper, emparer, happer, ramasser, obtenir, approprier, confisquer, enlever, collecter, récupérer];
   : verb aliases = [dejar, drop, laisser, quitter, poser, jeter, déposer];   
   : verb aliases = [aide, help, ayuda, hint, indice, indication, piste, manuel, instruction, guide];
}
```

## Pronominal verbs

Some players might want to use pronominal verbs, for example SE LEVER. The workaround here is to rewrite the sentence so we remove the “SE”.
For this, in each location, you can call the subroutine

```
on_command {
      : gosub "sentence_rewriting";
	 More code
}
```

And the subroutine can be like this, where you can also take advantage and do some other preprocessing of the player’s introduced command

```
subroutines {
   sentence_rewriting : subroutine {
      // transform look into examine if it is look something
      : if (verb_is "regarder" && !noun1_is "") {
         : set_sentence "examiner $1";
      }
      // pronominal verbs
      : if (verb_is "me" || verb_is "te" || verb_is "se" || verb_is "nous" || verb_is "vous") {
         : if (noun1_is "regarder" || noun1_is "examiner") {
            : set_sentence "examiner moi";
            // (noun1_is "tirer" || !noun2_is "") did not work so we check the exact sentence
         } : else_if (noun1_is "casser" || sentence_raw() == "me tirer" || sentence_raw() == "te tirer" || sentence_raw() == "se tirer" || sentence_raw() == "nous tirer" || sentence_raw() == "vous tirer") {
            : set_sentence "sortir";
            // for the case of "se tirer une balle"
         } : else_if (noun1_is "tirer") {
            : set_sentence "mourir";
         } : else {
            : set_sentence "$1 $2";
         }
      }
   }
}
```

When the verb starts with vowel, for example S’EXAMINER, this will not work, but you can still match it in your on_command code, ignoring the ‘, for instance using
`: match " sexaminer _; mexaminer _" {  }`

## Game restart or exit
Game restart
FIN is the spanish command to quit the game, which actually is like restarting the game. With the default behaviour of Adventuron the player will be asked if they want to quit but it expects "s" for Yes or "n" for No. The workaround is to create a synonym

```
   : verb aliases = [fin, recommencer, redémarrer, redémarrage, restart, quit];
```

and then, in the main on_command you can add

```
   : match "fin -" {
      : add_choice "Oui"  {
         : lose_game;
      }
      : add_choice "No"  {
         : done;
      }
      : choose "<Souhaitez-vous recommencer le jeu ?<6>>" ;
   }
```

## Articles

When the noun starts with vowel, like "arbre", or sounds like a vowel "homme", the article is "l'arbre" or "l'homme". When the player introduces "l'arbre", Adventuron will automatically translate it to "larbre" ignoring the ' symbol. The workaround is to add a synonym if the noun is used in several places
```
   : noun aliases = [arbre, larbre];
   : noun aliases = [ennemi, lennemi];
```

or repeating it in the match command if the noun is not used several times

```
: match "grimper arbre; grimper larbre" {

}
```

Of course, this can led to recognizing strange sentences like "grimper le larbre" but it will allow players to use "l'arbre" while playing.

You can also add
```
   : noun aliases = [e, est, lest];
   : noun aliases = [o, ouest, louest];
```
if you want to allow players to write things like "aller à l'est"

## Popup menu in desktop mode
In desktop mode, when you right click the screen, a pop-up menu appears which is written in Spanish. You can try to translate it directly in the generated html or you can just deactivate it. For this you should add this in the generated html just after the other existing <style> … </style> entry. (Thanks eツ for this code)

`<style>div.menu-popup.advpopuproot{display:none}</style>`

## System messages
In your theme you can define the system messages. This is a proposal for the translation to French for the system messages that can be edited in your game.

```
themes {
   my_theme : theme {
      system_messages {
         transcript_start_message = D'accord
         all_treasures_found_win_game                   = Félicitations, vous avez trouvé tous les trésors. Vous avez gagné !
         already_in_container                           = ${entity} est déjà à l'intérieur du ${entity2}.
         ask_new_game                                   = Souhaitez-vous commencer un nouveau jeu ?
         ask_quit                                       = Souhaitez-vous quitter le jeu ?
         be_more_specific                               = Sois plus précis ...\s
         cannot_carry_any_more                          = Vous avez les mains pleines.
         cannot_carry_any_more_weight                   = C'est trop lourd à porter.
         cant_get_character                             = Pas la meilleure des idées.
         cant_get_scenery                               = Vous ne pouvez pas le prendre.
         cant_see_one_of_those                          = Vous ne pouvez pas en voir un.
         dont_have_one_of_those                         = Vous n'en avez pas un !
         exit_list_additional_exits_are_located_verbose = Des sorties supplémentaires se trouvent\s
         exit_list_end_text                             = .
         exit_list_end_text_verbose                     = .
         exit_list_from_here_you_can_go_verbose         = Vous pouvez aller\s
         exit_list_header_concise                       = Sorties :\s
         exit_list_last_sep_verbose                     = \set\s
         exit_list_sep_verbose                          = ,\s
         exit_list_there_are_no_obvious_exits           = Il n'y a pas de sorties évidentes.
         exit_list_to_the_verbose                       = vers le
         exit_list_you_can_also_go_verbose              = Vous pouvez aussi aller\s
         gamebook_question                              = Sélectionnez une option ...
         i_cant_do_that                                 = Vous ne pouvez pas faire ça.
         invalid_choice                                 = Choix invalide.
         inventory_list_empty                           = Rien
         inventory_list_end_text                        = .
         inventory_list_final_separator                 = \set\s
         inventory_list_header                          = Vous portez:\s
         inventory_list_header_verbose                  = Vous portez\s
         inventory_list_separator                       = ,\s
         it_is_dark                                     = C'est sombre. Vous ne pouvez rien voir.
         must_remove_first                              = Essayez d'abord de l'enlever.
         not_carried                                    = Vous ne pouvez pas ${verb} quelque chose que vous ne portez pas.
         not_present                                    = ${entity} n'est pas là.
         nothing_here                                   = Il n'y a rien ici.
         nothing_to_get                                 = Vous ne le trouvez pas.
         object_list_empty                              = Rien
         object_list_end_text                           = .
         object_list_final_separator                    = \set\s
         object_list_header                             = Vous voyez:\s
         object_list_header_after_initial               = Vous voyez également:\s
         object_list_header_verbose                     = Vous voyez\s
         object_list_header_verbose_after_initial       = Vous voyez également\s
         object_list_separator                          = ,\s
         ok                                             = OK
         on_drop                                        = Vous déposez ${entity}.
         on_get                                         = Vous prenez ${entity}.
         on_put                                         = Vous mettez ${entity} dans ${entity2}.
         on_put_non_container                           = ${entity} n'est pas un conteneur.
         on_put_non_surface                             = ${entity} n'est pas une surface.
         on_remove                                      = Vous enlevez ${entity}.
         on_wear                                        = Vous portez ${entity}.
         post_quit                                      = Vous avez quitté le jeu.
         prompt                                         = >>
         prompt_prefix                                  = Et maintenant?
         question_prompt_char                           = ?
         there_is_nothing_you_can                       = Il n'y a rien que vous puissiez ${verb} pour le moment.
         treasure_message                               = 
         treasure_suffix                                = 
         tutorial_message_prefix                        = Tutoriel:\s
         unknown_noun                                   = Nom inconnu - "${noun}".
         unknown_verb                                   = Verbe inconnu - "${verb}".
         verb_noun_only                                 = Ce jeu ne nécessite que deux entrées de mots.
         worn_suffix                                    = \s(porté)
         you_already_wear                               = Vous portez déjà ça.
         you_are_already_carrying                       = Vous avez déjà ${entity}.
         you_are_not_holding                            = Vous ne tenez pas ${entity}.
         you_cant_go_that_direction                     = Vous ne pouvez pas aller par là.
         you_cant_wear                                  = Vous ne pouvez pas porter ça.
         you_cant_wear_anything_else                    = Vous ne pouvez pas porter autre chose sans enlever quelque chose au préalable.
         you_dont_wear                                  = Vous ne portez pas ça.
         you_see_nothing_special                        = Vous ne voyez rien de spécial.
         you_see_nothing_special_2                      = Vous ne pouvez pas interagir avec cela.
      }
   }
}
```

## Other system messages

Some system messages are not contained in the list of system messages you can directly edit in your game. If you still want to modify them, you will have to modify them each time you compile your game in html. For instance, the messages to Save or Load are not part of the system messages and if your game allows saving and loading then you will have to replace these messages in the html.

This can be annoying so maybe you should create a script to do it, as when you compile the game again from the Adventuron editor, all these changes will have to be manually repeated…

For example:

`Pantalla Completa` -> Plein écran

Also notice that accents are not displayed in the html strings. For example vacío, which means empty (for an empty slot to save the game) appears in the html as vac\xEDo

`vac\xEDo` -> vide

