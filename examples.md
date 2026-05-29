# StablePool Examples

## Basic Object Management

```clojure
(use StablePool)

(defn main []
  (let [pool (StablePool.new)]
    (do
      ;; Allocate objects
      (let [h1 (StablePool.alloc! &pool 100)
            h2 (StablePool.alloc! &pool 200)]
        (do
          (IO.println &(str (StablePool.get &pool &h1))) ; (Just 100)
          
          ;; Free an object
          (StablePool.free! &pool &h1)
          
          ;; h1 is now stale
          (IO.println &(str (StablePool.get &pool &h1))) ; Nothing
          
          ;; Next allocation will likely reuse h1's slot but with a new generation
          (let [h3 (StablePool.alloc! &pool 300)]
             (IO.println &(str (StablePool.get &pool &h3)))) ; (Just 300)
        )))))
```

## Entity System Integration

StablePool is ideal for ECS-like architectures where entities need stable references.

```clojure
(use StablePool)
(use Handle)

(deftype Entity [
  name String
  hp Int
])

(defn main []
  (let [entities (StablePool.new)]
    (do
      (let [player (StablePool.alloc! &entities (Entity.init @"Player" 100))
            enemy  (StablePool.alloc! &entities (Entity.init @"Goblin" 20))]
        (do
          ;; Process active entities
          (StablePool.doall &(fn [e] (IO.println (Entity.name e))) &entities)
          
          ;; Safe deletion
          (StablePool.free! &entities &enemy)
          
          ;; Other systems holding the 'enemy' handle can safely detect it's gone
          (match (StablePool.get &entities &enemy)
            (Maybe.Just e) (IO.println "Still here")
            (Maybe.Nothing) (IO.println "Enemy defeated!"))
        )))))
```
