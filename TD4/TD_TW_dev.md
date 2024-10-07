# Tailwind avancé

## 1/ Limitations

### 1_1 Etat par defaut :

```javascript

function Button({ children, ...restProps }) {
  return (
    <button
      {...restProps}
      className="rounded-lg bg-emerald-600 px-5 py-2 font-semibold text-white hover:bg-emerald-500 active:bg-emerald-700"
    >
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div className="grid h-screen place-content-center">
      <Button className="">Click me</Button>
    </div>
}
  );

```

### 1_2 Ajout d'une props `className` :

- 🚀 Ajouter la classe `bg-blue-500` dans le composant `Button` à l'aide de la props `className`. Qu'observez-vous ?

Ici l'ajout d'une classe `bg-blue-500` dans le composant `Button` ne fonctionnera pas car la props `className` est définie avant et sera écrasée par l'attribut `className` du composant.

- 🚀 Déplaçons `restProps` à la fin. Qu'observez-vous ?

Ici l'ensemble des classes de l'attribut `className` de `Button` sont écrasés par la props `className`.

### 1_3 Concaténation des classes Tailwind avec un template literal :

```javascript
// Dans le composant Button
...
className={`rounded-lg bg-emerald-600 px-5 py-2 font-semibold text-white hover:bg-emerald-500 active:bg-emerald-700 ${className}`}
...
```

- 🚀 Concaténation des classes à l'aide du template littéral. Qu'observez-vous ?

Nous avons un conflit entre `bg-emerald-600` et `bg-blue-500`.

- Relancer le serveur. Qu'observez-vous ?

- Idem testez avec `bg-red-500`.

💡 **Remarque** : Les conflits entre les classes Tailwind - apparaissentt avec l'extension _Tailwind CSS IntelliSense_.

⚠️ **Attention** au mode de création des classes dans Tailwind : 'just-in-time compiler' : l'order des classes Tailwind compilées est un 'ordre alphabétique' et non l'ordre de déclaration.

---

## 2/ TailwindMerge

**TailwindMerge** est un outil qui vous permet de fusionner plusieurs classes CSS Tailwind et résout automatiquement les conflits entre les classes en supprimant les classes en conflit avec une classe définie plus tard. C'est particulièrement utile lorsque vous souhaitez remplacer les classes CSS Tailwind dans vos composants.

https://www.npmjs.com/package/tailwind-merge
https://github.com/dcastil/tailwind-merge/blob/v2.0.0/docs/features.md

- Installation

```bash
npm install tailwind-merge

```

- Utilisation

Utilisation simple :

```javascript
// Dans le composant Button
import {twMerge} from "tailwind-merge";

...
className = {twMerge(
  "rounded-lg bg-emerald-600 px-5 py-2 font-semibold text-white hover:bg-emerald-500 active:bg-emerald-700",
  className)}
...
```

Utilisation avec condition :

```javascript
// Dans le composant Button
...
className = {twMerge("rounded-lg bg-emerald-600 px-5 py-2 font-semibold text-white hover:bg-emerald-500 active:bg-emerald-700",
  className,
  active && "bg-gray-600")}
...
```

---

## 3/ Clsx et cn()

Un autre outil - **clsx** - **Rendu de classe conditionnelle**, vous permet de définir conditionnellement des noms de classes react en utilisant un objet ou une liste de chaînes.

https://www.npmjs.com/package/clsx

- Installation

```bash
npm install clsx
```

- Fonction cn()
  ⚠️ Clsx ne résout pas les conflits entre les classes comme **TailwindMerge**.
  💡 Nous allons donc combiner `clsx` et `tailwind-merge` avec la `fonction cn()`

```javascript
// Créer un répertoire libs dans src et créer un fichier utils.js
import { twMerge } from "tailwind-merge";
import { clsx } from "clsx";

export function cn(args) {
  return twMerge(clsx(...args));
}
```

- Utilisation

```javascript
// Dans le composant Button
// Importer la fonction cn() dans le composant Button
...
className = {cn(
  "rounded-lg bg-emerald-600 px-5 py-2 font-semibold text-white hover:bg-emerald-500 active:bg-emerald-700",
  className,
  {"bg-gray-600": active})
  }
```

---

## 4/ Variante de classe

### 4_1 Méthode 1

```javascript
const baseClasses = "rounded-md font-medium focus:outline-none";

const variantsLookup = {
  primary:
    "bg-cyan-500 text-white shadow-lg hover:bg-cyan-400 focus:bg-cyan-400 focus:ring-cyan-500",
  secondary:
    "bg-slate-200 text-slate-800 shadow hover:bg-slate-300 focus:bg-slate-300 focus:ring-slate-500",
  danger:
    "bg-red-500 text-white shadow-lg uppercase tracking-wider hover:bg-red-400 focus:bg-red-400 focus:ring-red-500",
  text: "text-slate-700 uppercase underline hover:text-slate-600 hover:bg-slate-900/5 focus:text-slate-600 focus:ring-slate-500",
};

const sizesLookup = {
  small: "px-3 py-1.5 text-sm focus:ring-2 focus:ring-offset-1",
  medium: "px-5 py-3 focus:ring-2 focus:ring-offset-2",
  large: "px-8 py-4 text-lg focus:ring focus:ring-offset-2",
};

export default function Button = ({ variant = "primary", size = "medium", ...rest }){
  return (
    <button
      {...rest}
      className={`${baseClasses} ${variantsLookup[variant]} ${sizesLookup[size]}`}
    />
  );
};
```

### 4_2 Méthode 2 : Class Variance Autority CVA

- Documentation

https://www.npmjs.com/package/class-variance-authority

https://cva.style/docs/examples/react/tailwind-css

- Installation

```bash
npm i class-variance-authority
```

⚠️ Pour que _Tailwind CSS IntelliSense_ continue à fonctionner, ajoutez au fichier `settings.json`

```json
{
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ]
}
```

- Utilisation

```javascript
import {cn} fron "lib/utils";
import { cva } from "class-variance-authority";
const base = "mes classes de base";

const button = cva(base, {
  variants: {
    intent: {
      primary: "",
      secondary: "",
      alert: "",
    },
    size: {
      small: "",
      medium: "",
    },
    rounde: {
      rd: "",
      nrd: "",
    },
  },
  compoundVariants: [],
  defaultVariants: {
    intent: "primary",
    size: "medium",
  },
});

export default function ButtonCVA({
  className,
  intent,
  size,
  rounde,
  ...props
}) {
  return (
    <button
      {...rest}
      className={cn(
        baseClasses,
        variantsLookup[variant],
        sizesLookup[size],
        className
      )}
    />
  );
}
```

---

## 5 ShadCn UI :

Build your component library
Beautifully designed components that you can copy and paste into your apps.

**Shadcn UI**, une 'bibliothèque' de composants graphiques basés sur Radix UI et Tailwind CSS
https://ui.shadcn.com
Les composants sont conçus pour être utilisés dans vos applications Web et sont entièrement personnalisables.

Exercice :

- 🚀 Créer un composant `Button` avec `Shadcn UI` et personnaliser le composant avec les variantes de classe.
- Choisir le design system d'une marque (Decathlon, Orange, Spotify, etc.) et créer une composant.
  cf : https://designsystems.fr, https://adele.uxpin.com, https://designsystemsrepo.com/design-systems/, https://medium.com/pelostudio/le-top-des-design-system-en-2023-c621f50bef6e
- Créer une mini application ToDoList avec les composants `Shadcn UI` précédement réalisés.

Configuration : https://ui.jln.dev
