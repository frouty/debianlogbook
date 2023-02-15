Regex
#####


https://www.grymoire.com/Unix/Regular.html

Repetions des caracteres avec *
*******************************

   - "[0-9]" match *zero ou plusieurs* nombres
   - "[0-9]\*" match *un  ou plusieurs* nombre

   - "^ \*" match *zero ou plusieurs* espace au debit d'une ligne.

On peut utiliser cette technique pour spécifier le nombre minimum de répétion. On ne peut spécifier le nombre maximum de répétiion avec *

Minimum et maximun nombre de répétitions
****************************************

Normalement un backslash \ annule le sens des characteres spéciaux, eg: \. pour avoir un point.

Mais si un \\ est placé avant certains caractéres:
   - <
   - >
   - {
   - {
   - (
   - )
   - avant un nombre 
   
Le backshlash va donner un nouvelle fonction. 

[a-z]\\{4,8\\} match mot minuscule de 4 , 5 , 6 , 7 , 8 lettres

[a-z]\\{4,\\}  match mots minuscules de 4 lettres ou plus .

[a-z]\\{4\\}  match mots minuscules de 4 lettres.

\\+ match une ou plus

\\? match zero ou une occurence

\\n match le new line caractere

'.*' match toutes les strings meme un string vide

'.\\+' match toutes les strings qui contiennent au moins 1 caratere.

'^# ' match une string commencant par #


