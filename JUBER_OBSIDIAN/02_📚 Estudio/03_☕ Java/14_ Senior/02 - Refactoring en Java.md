---
tags:
  - Java
  - Refactoring
Creado: 2026-05-21
Relacionado:
  - Senior Java Engineer
---

# Refactoring en Java

Refactorizar es cambiar estructura sin cambiar comportamiento observable.

## Tecnicas frecuentes

| Tecnica | Uso |
|---|---|
| Extract Method | Bajar complejidad |
| Extract Class | Separar responsabilidades |
| Introduce Parameter Object | Reducir parametros |
| Replace Conditional with Polymorphism | Evitar if gigantes |
| Encapsulate Collection | Proteger invariantes |

## Antes de refactorizar

- Agrega tests si el comportamiento no esta cubierto.
- Haz cambios pequenos.
- Corre tests seguido.
- No mezcles refactor con feature grande si no hace falta.

## Ejemplo

```java
public Money total() {
    return lines.stream()
        .map(OrderLine::subtotal)
        .reduce(Money.zero(), Money::add);
}
```

## Checklist

- [ ] El comportamiento se mantiene.
- [ ] El codigo resultante expresa mejor la intencion.
- [ ] No cree abstracciones prematuras.
- [ ] Los tests siguen pasando.

---

## ✍️ Mis apuntes
<!-- Tus dudas, ejemplos propios, cómo lo aplicaste -->

