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

Te devuelve el elemento del array que haga match con la key y el value que le pasas por parámetro:

```
const findByKey = (array, key, value) => {
  for (let i = 0; i < array.length; i++) {
    if (array[i][key].toLowerCase() === value.toLowerCase()) {
      return array[i];
    }
  }
  return null;
};
```

- Modo de uso:

```
const arr2 = [
  {
    name: "Luis",
    lastName: "Gonzalez",
  },
  {
    name: "David",
    lastName: "Jimenez",
  },
  {
    name: "Juan",
    lastName: "Perez",
  },
];

findByKey(arr2, "name", "Juan"); // { name: 'Juan', lastName: 'Perez' }

```


Te devuelve los elementos del array que haga match con la key y el value que le pasas por parámetro:

```
const findItemsByKey = (array, key, value) => {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    if (array[i][key].toLowerCase() === value.toLowerCase()) {
      result.push(array[i]);
    }
  }
  return result;
}
```

- Modo de uso:

```
const arr2 = [
  {
    name: "Luis",
    lastName: "Gonzalez",
  },
  {
    name: "David",
    lastName: "Jimenez",
  },
  {
    name: "Juan",
    lastName: "Perez",
  },
  {
    name: "Juan",
    lastName: "Perez",
  },
];

findItemsByKey(arr2, "name", "Juan"); // [{ name: 'Juan', lastName: 'Perez' }, { name: 'Juan', lastName: 'Perez' }]

```



# Función para generar traducciones desde un csv en node

Se requieren la dependencia ```npm i csv-parser```

```
const csv = require("csv-parser");
const fs = require("fs");

// Para generar algún error en caso de que sea necesario
const throwWarning = (require = false, message = "") => {
  if (require) throw new Error(message);
};

// Para crear un objeto json a partir de unas keys dadas, el valor y un objeto previo

const createJsonObject = (key = "", value = "", source = {}) => {
  const arrKeys = key.split(".");
  const firstKey = arrKeys.shift();
  const { [firstKey]: newSource = {} } = source;

  if (arrKeys.length > 0) {
    return {
      [firstKey]: {
        ...(newSource || {}),
        ...createJsonObject(arrKeys.join("."), value, newSource),
      },
    };
  }

  return {
    ...(newSource || {}),
    [firstKey]: value,
  };
};


// Para vberificar si es un csv válido
const checkIfIsCsvFile = (
  sourcePath = throwWarning(true, "sourcePath argument is required")
) => {
  if (!fs.existsSync(sourcePath)) {
    return {
      error: true,
      message: `[${sourcePath}] file does not exist`,
    };
  }

  const file = fs.readFileSync(sourcePath);
  const fileContent = file.toString();
  const isSplittedByComma = fileContent.split(",").length > 1;

  if (!fileContent.includes("\n")) {
    return {
      error: true,
      message: `[${sourcePath}] file is not a csv file`,
    };
  }

  if (!isSplittedByComma) {
    return {
      error: true,
      message: `[${sourcePath}] is not splitted by comma`,
    };
  }

  return {
    error: false,
    message: `[${sourcePath}] is a csv file`,
  };
};


// Función para crear el csv
const generateObjFromCsv = (
  sourcePath = throwWarning(true, "sourcePath argument is required")
) => {
  const { error, message } = checkIfIsCsvFile(sourcePath);

  if (error) {
    return throwWarning(true, message);
  }

  fs.createReadStream(sourcePath)
    .pipe(csv())
    .on("data", (data) => {
      const { key, ...rest } = data;
      const langs = [...new Set(Object.keys(rest))];

      langs.forEach((lang) => {
        const translationsObj = require(`../src/translations/${lang}/${lang}.json`);
        const jsonObj = {
          ...translationsObj,
          ...createJsonObject(key, rest[lang], translationsObj),
        };

        fs.writeFileSync(
          `src/translations/${lang}/${lang}.json`,
          JSON.stringify(jsonObj, null, 2)
        );
      });
    });
};


```


# Patrón de diseño compound-component
```
import { FC, PropsWithChildren } from "react";
import { Navbar } from "./Navbar/Navbar";
import { Content } from "./Content/Content";
import { Sidebar } from "./Sidebar/Sidebar";
import { Layout as LayoutHOC } from "./Layout/Layout";
export * from "./Layout/Layout";
export interface LayoutHocProps<T = {}> {
  (props: PropsWithChildren<T>): JSX.Element | null;
  Navbar: FC<PropsWithChildren<T>>;
  Content: FC<PropsWithChildren<T>>;
  Sidebar: FC<PropsWithChildren<T>>;
}

const Layout: LayoutHocProps = Object.assign(LayoutHOC, {
  Navbar,
  Content,
  Sidebar,
});

export default Layout;


<Layout>
      <Layout.Navbar />
      <Layout.Sidebar />
      <Layout.Content />
</Layout>
```


# Mix de javascript (ES6)

Sirve para cambiar de posiciones los elementos dentro de un array:

```
const arr = [5,8];
[arr[0], arr[1]] = [arr[1], arr[0]];
console.log(arr)
Salida: [8, 5]
```



# Generar getters y setters con decoradores

```
import { capitalize } from "lodash";

const Getters = () => <T extends {new(...args:any[]):{}}>(constructor:T) => {
  return class extends constructor {
    constructor(...args: any[]) {
      super(...args);
      const props = Reflect.ownKeys(this);
      props.forEach((prop: string) => {
        const capitalizedKey = capitalize(prop);
        const methodName = `get${capitalizedKey}`;
        Object.defineProperty(this, methodName, { value: () => this[prop] });
      });
    }
  }
}
const Setters = () => <T extends {new(...args:any[]):{}}>(constructor:T) => {
  return class extends constructor {
    constructor(...args: any[]) {
      super(...args);
      const props = Reflect.ownKeys(this);
      props.forEach((prop: string) => {
        const capitalizedKey = capitalize(prop);
        const methodName = `set${capitalizedKey}`;
        Object.defineProperty(this, methodName, { value: (newValue: any) => { this[prop] = newValue } });
      });
    }
  }
}

@Getters()
@Setters()
export class Person {
  [x: string]: any;
  nom: string;
  prenom: string;

  constructor(nom: string, prenom: string) {
    this.nom = nom;
    this.prenom = prenom;
  }
}
```


# Sobrecarga de funciones

Este es un ejemplo de sobrecarga de funciones en TypeScript.

```
function loMismo(param: string): string;
function loMismo(param: number): number;
function loMismo(param: any): any {
  return param;
}

const result = loMismo("hola"); // result: string
const result2 = loMismo(2); // result: number
```

La función loMismo está definida tres veces: dos veces con una firma que especifica el tipo de parámetro y el tipo de valor de retorno, y una vez con una implementación genérica que toma cualquier tipo de parámetro y devuelve el mismo tipo de valor.

La idea detrás de la sobrecarga de funciones es permitir que una función acepte diferentes tipos de parámetros y proporcione una respuesta apropiada para cada tipo de entrada. En este caso, la función loMismo puede tomar una cadena o un número como entrada y simplemente devolver esa entrada sin modificarla.
