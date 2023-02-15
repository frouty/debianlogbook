Password generator
##################

::
   
   dd if=/dev/random bs=128 count=1 | tr -dc '\!-\~' | head -c 32 ; echo ''

