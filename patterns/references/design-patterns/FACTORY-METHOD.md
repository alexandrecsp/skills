# Factory Method

## Overview

Subclasses of a **creator** override its **factory method** to return different products behind one shared interface, so code that uses the product never names a concrete class. The payoff is adding a new product type without editing the code that consumes it, and one place where creation lives.

## When to Use

- The exact product type, or its dependencies, are unknown until runtime or belong to the subclass (a logistics app that starts with trucks and later ships boats)
- Creation should be extendable without touching the code that uses the product
- A library or framework lets users extend its internal components by subclassing
- Creation should reuse existing objects (a pool, a cache) rather than always building new ones

## When Not to Use

- A `new` of one concrete class that never varies needs no factory
- A plain function that returns the right product by key (a *simple factory*) is often enough when no creator hierarchy exists; Factory Method earns its subclasses when one already exists or is coming, with more than one creator subclass in view
- A registry or direct `new` that already covers the variation makes the pattern ceremony

## Core Principles

### 1. Products share one interface

The creator and its clients rely on the product interface only. If the products cannot honour one interface, the abstraction is wrong. Settle the interface first.

### 2. The creator's job is business logic; creation is a hook

The creator is not "a factory class"; it has real work that uses a product, and the factory method is the seam that work calls. Subclasses override the seam, the surrounding logic stays shared.

```typescript
interface Transport { deliver(cargo: Cargo): void; }

abstract class Logistics {
  planDelivery(cargo: Cargo) {
    const transport = this.createTransport(); // factory method
    transport.deliver(cargo);
  }
  protected abstract createTransport(): Transport;
}

class RoadLogistics extends Logistics {
  protected createTransport(): Transport { return new Truck(); }
}
class SeaLogistics extends Logistics {
  protected createTransport(): Transport { return new Ship(); }
}
```

### 3. Every construction site calls the factory method

Replace each direct `new Concrete()` with the factory method call. One remaining direct construction reintroduces the coupling and the scattered conditionals.

### 4. Choose the creator once, at the edge

The client or composition root decides which concrete creator to instantiate (from config, environment, a request field) and passes it in. Creator selection stays out of the logic that uses products.

### 5. Many product types: a parameter, a registry, or a bigger pattern

A factory method may take a parameter that selects among products, but a `switch` that grows with each new product signals the next step: a registry keyed by type, **Abstract Factory** (families of related products), **Builder** (stepwise construction of complex ones), or **Prototype** (cloning).

### 6. Default behaviour is optional

Make the base factory method abstract when no sensible default exists; give it a default implementation when most creators share one product. Pooling or caching is a legitimate body for the method: return an existing instance when one fits.

## Verification

- [ ] The variation is named: what differs between products and who decides
- [ ] All products implement one interface; clients use only that and never `instanceof`-check a product
- [ ] The creator contains real logic that calls the factory method
- [ ] No concrete product is constructed outside a factory method
- [ ] The concrete creator is chosen in one place, outside the business logic
- [ ] The factory method's `switch` is not growing with each new product; if it is, a registry or bigger pattern replaced it