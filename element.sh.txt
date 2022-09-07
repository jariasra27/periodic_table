#!/bin/bash
PSQL="psql -X --username=freecodecamp --dbname=periodic_table --tuples-only -c"
FORMATED_OUTPUT() {
  echo $1 | while read ATOMIC_NUMBER BAR SYMBOL BAR NAME BAR MASS BAR MELTING_POINT BAR BOILING_POINT BAR TYPE
  do
    echo "The element with atomic number $ATOMIC_NUMBER is $NAME ($SYMBOL). It's a $TYPE, with a mass of $MASS amu. $NAME has a melting point of $MELTING_POINT celsius and a boiling point of $BOILING_POINT celsius."
  done
}
CHECK_ATOMIC_NUMBER() {
  if [[ $1 =~ ^[0-9]+$ ]]
  then
    RESULT=$($PSQL "SELECT e.atomic_number, e.symbol, e.name, p.atomic_mass, p.melting_point_celsius, p.boiling_point_celsius, t.type
    FROM elements AS e INNER JOIN properties AS p USING(atomic_number) INNER JOIN types AS t USING(type_id) WHERE atomic_number = $1")
    if [[ -z $RESULT ]]
    then
      echo "I could not find that element in the database."
    else
      FORMATED_OUTPUT "$RESULT"
    fi
  else
    echo "I could not find that element in the database."
  fi
}
CHECK_NAME() {
  RESULT=$($PSQL "SELECT e.atomic_number, e.symbol, e.name, p.atomic_mass, p.melting_point_celsius, p.boiling_point_celsius, t.type
   FROM elements As e INNER JOIN properties AS p USING(atomic_number) INNER JOIN types AS t USING(type_id) WHERE name = '$1'")
  if [[ -z $RESULT ]]
  then
    CHECK_ATOMIC_NUMBER $1
  else
    FORMATED_OUTPUT "$RESULT"
  fi
}
CHECK_SYMBOL() {
  RESULT=$($PSQL "SELECT e.atomic_number, e.symbol, e.name, p.atomic_mass, p.melting_point_celsius, p.boiling_point_celsius, t.type 
  FROM elements AS e INNER JOIN properties AS p USING(atomic_number) INNER JOIN types AS t USING(type_id) WHERE symbol = '$1'")
  if [[ -z $RESULT ]]
  then
    CHECK_NAME $1
  else
    FORMATED_OUTPUT "$RESULT"
  fi
}

if [[ -z $1 ]]
then
  echo "Please provide an element as an argument."
else
  CHECK_SYMBOL $1
fi

