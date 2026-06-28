```ts
/**
 * Извлекает объединение типов значений указанного ключа из массива объектов.
 * @template {readonly unknown[]} ArrayType - массив объектов (только для чтения)
 * @template {string} ElementKey - ключ объекта
 */
type UnionOfObjectValues<
  ArrayType,
  ElementKey extends string,
> = ArrayType extends readonly (infer ElementType)[]
  ? ElementType extends Record<string, unknown>
    ? ElementType[ElementKey]
    : never
  : never;
```

Пример с `Vue.js`: тип `UnionOfObjectValues` может быть использован при работе с компонентом "выпадающий список" для объявления возможных вариантов модели.

```vue
<script lang="ts" setup>
import { reactive } from "vue";

const SELECT_OPTIONS = [
  { label: "Первый элемент", value: "first-option" },
  { label: "Второй элемент", value: null },
  { label: "Третий элемент", value: 3 },
] as const;

interface State {
  model: UnionOfObjectValues<typeof SELECT_OPTIONS, "value">;
}

const state: State = reactive({
  model: "first-option",
});
</script>

<template>
  <VSelect v-model="state.model" :options="SELECT_OPTIONS" />
</template>
```

Таким образом, модель будет иметь строго определённое объединение типов: `"first-option" | null | 3`, которое не придётся составлять вручную. Стоит также обратить внимание на утверждение константности массива SELECT_OPTIONS через `as const`. Без этого утверждения варианты модели будут менее строгими (более общими): `string | null | number`.

---

```ts
/**
 * Рекурсивно применяет модификатор readonly ко всем вложенным свойствам объектов
 * и элементам массивов, гарантируя неизменяемость структуры на всех уровнях.
 * @template T Тип объекта, массива или примитива
 */
type DeepReadonly<T> =
  T extends ReadonlyArray<infer U>
    ? ReadonlyArray<DeepReadonly<U>>
    : T extends object
      ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
      : T;
```

---

```ts
/**
 * Делает все свойства T доступными только для чтения, кроме тех, чьи ключи указаны в K.
 * Преобразование неглубокое.
 * @template T Исходный объект
 * @template K Ключи свойств объектов, которые не будут помечены модификатором readonly
 */
type PartialReadonly<T, K extends keyof T> = Readonly<Omit<T, K>> & Pick<T, K>;
```

Пример с `Vue.js`: тип `PartialReadonly` может быть полезен в тех случаях, когда в дочерний компонент передаётся объект данных и при этом имеется потребность в защите от изменений значений большинства свойств данного объекта.

```vue

<script lang="ts" setup>
interface Request {
  // Свойства, которые должны быть доступны только для чтения
  group: string
  externalId: number
  regionId:number
  brand: string
  articleCode: string
  sortBy: string

  // Свойства, значения которых можно изменять
  isAvailable: boolean
  productType: string
}

interface ProductRequest extends PartialReadonly<Request, 'isAvailable' | 'productType'> {
  // Новое свойство с доступным для изменения значением
  name: string
}

const request = defineModel<ProductRequest>({ required: true })

// Изменение свойств без ошибок от компилятора TypeScript
request.value.isAvailable = true
request.value.productType = 'electronics'
request.value.name = 'headphones'

// Компилятор TypeScript обозначит ошибку: Cannot assign to 'group' because it is a read-only property.
request.value.group = 'regional'


```

---

```ts
/**
 * Делает все свойства T доступными только для чтения, кроме тех, чьи ключи указаны в K.
 * Преобразование глубокое.
 * @template T Исходный объект
 * @template K Ключи свойств объектов, которые не будут помечены модификатором readonly
 */
type PartialReadonly<T, K extends keyof T> = DeepReadonly<Omit<T, K>> &
  Pick<T, K>;
```
