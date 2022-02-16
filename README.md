# Typescript nivel avanzado

## Alias

**Un alias es para renombrar un tipado complejo que se repite muchas veces.**

Se puede poner un alias en tipos sencillos tales como:

```
type Message = string;

const mensaje: Message = "Hola mundo";

type FunctionVoid = () => void;

const func: FunctionVoid = () => {};

type Whatever<T> = {
  value: T;
};

const whayever: Whatever<String> = {
  value: "Hola mundo",
};
```

Esto nos facilita mucho para los reducers

```
type ReducerFunction<S> = (prevState: S, update: Partial<S>) => S;

interface State {
  user: null | Message;
}

const reducer: ReducerFunction<State> = (
  prevState: State,
  update: Partial<State>
) => {
  return { ...prevState, ...update };
};
```

## Intersección

Sirve para convinar múltiples tipos en uno solo

```
const compose = <A, B>(a: A, b: B): A & B => ({ ...a, ...b });

const cat = compose({ type: "Feline" }, { skill: "hunting" });

console.log(cat); // {type: "Feline",skill:"hunting" }

```

## Mapped types

```
interface ProductItem {
  name: string;
  price: number;
}

type Evolver<O> = {
  [Key in keyof O]: (args: O[Key]) => O[Key];
};

const formatString = (str: string) => (
  (str = str.trim()), str[0].toUpperCase() + str.substr(0)
);

const applyIva = (price: number): number => price * 1.21;

const transformation: Evolver<ProductItem> = {
  name: formatString,
  price: applyIva,
};

```

## Tipos condicionales

Sirve para devolver un tipo de dato condicionalmente

```
type DarkColors = "black" | "grey";
type LightColors = "white" | "yellow" | "pink";

type Status = "sad" | "happy";

type Pallete<T extends Status> = T extends "sad" ? DarkColors : LightColors;
```

---

# TIPOS GENÉRICOS DE UTILIDADES

## Partial

Nos permite convertir en opcionales las propiedades de una interfaz.

```
interface Person {
    name: string;
    age: number;
}

type PartialPerson = Partial<Person>;
/**
 * type PartialPerson = {
    name?: string;
    age?: number;
}
*/
```

## required

Al contrario de partial, nos permite hacer obligatorias las propiedades de un objeto

```
interface Coords {
  x: number;
  y: number;
  z?: number;
}

type Coord3D = Required<Coords>;

/**
 * type Coord3D = {
    x: number;
    y: number;
    z: number;
}
 */

```

## Exclude y Extract

Sirve para extraer propiedades de un tipe

```
type WeekDay = "lun" | "mar" | "mie" | "jue" | "vie" | "sab" | "dom";
type WorkDay = Exclude<WeekDay, "sab" | "dom">;
 
 //  WorkDay typeof "lun" | "mar" | "mie" | "jue" | "vie" | "sab" | "dom"
 

type Weekend = Extract<WeekDay, "sab" | "dom">;
/**
 * Weekend typeof  "sab" | "dom"
 */

```


# Funciones avanzadas

Te devuelve cualquier propiedad de un objeto o elemento de un array

```
const getValue = (source, key) => {
  const arrKeys = key.split(".");
  const firstKey = arrKeys.shift();
  const { [firstKey]: newSource = undefined } = source;

  if (arrKeys.length > 0) {
    return getValue(newSource || {}, arrKeys.join("."));
  }

  return newSource;
};
```
- Modo de uso:
```

const obj = {
  name:'Lewis',
  lastname:'Lopez',
}

const arr = [
  {
    name:'Lewis',
    lastname:'Lopez',
  },
  {
    name:'Leinor',
    lastname:'Lopez',
  },
]


getValue(obj, 'name'); // Lewis

getValue(arr, '0'); // Fisrt item

getValue(arr, '0.name'); // Lewis



```
