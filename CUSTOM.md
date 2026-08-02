# MASpigot Custom API

Clases y eventos custom añadidos por MASpigot a la API de PandaSpigot (1.8.8).

---

## 🧱 Chunk Snapshots

### `us.mineaqua.spigot.chunk.DummyChunkSnapshot`

Marker interface para snapshots de chunk **restaurables** (a diferencia del `ChunkSnapshot` de Bukkit que es read-only).

### Métodos en `org.bukkit.Chunk`

| Método | Qué hace |
|---|---|
| `takeSnapshot()` | Captura bloques, **sin** tile entities |
| `takeSnapshot(boolean tileEntities)` | Captura bloques ± NBT de tile entities |
| `restoreSnapshot(DummyChunkSnapshot snap)` | Restaura bloques solamente |
| `restoreSnapshot(DummyChunkSnapshot snap, boolean tileEntities)` | Restaura bloques + tile entities |

```java
// Guardar y restaurar un chunk completo (cofres, hornos, etc.)
DummyChunkSnapshot saved = chunk.takeSnapshot(true);
// ... el chunk se modifica ...
chunk.restoreSnapshot(saved, true);
```

---

## 🥊 Knockback Profiles

### `us.mineaqua.spigot.option.Option` (interfaz)

```java
Option.simple("nombre", valorDefault);          // crear opción
Option.simple("nombre", valorDefault, valor);   // crear con valor inicial
option.getName();    // nombre
option.getValue();   // valor actual    (<T>)
option.setValue(v);  // cambiar valor
```

### `us.mineaqua.spigot.knockback.KnockbackProfile`

```java
KnockbackProfile(String name, Map<String, Option> options);
profile.getName();           // nombre del perfil
profile.getOptions();        // ImmutableList<Option>
profile.getOptionNames();    // ImmutableList<String>
profile.getOption("clave");  // Option por nombre
profile.getValue("clave");   // valor directo  (<T>)
```

### Métodos en `org.bukkit.entity.Player`

```java
player.getKnockbackProfile();              // perfil actual
player.setKnockbackProfile(profile);       // asignar (null = default)
```

```java
// Crear perfil de KB personalizado y asignarlo
Map<String, Option> opts = new HashMap<>();
opts.put("horizontal", Option.simple("horizontal", 0.4, 0.6));
opts.put("vertical",   Option.simple("vertical",   0.4, 0.5));

KnockbackProfile comboProfile = new KnockbackProfile("combo", opts);
player.setKnockbackProfile(comboProfile);

// Leer valor
double horiz = player.getKnockbackProfile().<Double>getValue("horizontal");
```

> Los perfiles se persisten en `knockback/` como YAML. Se gestionan con `/knockback`.

---

## 🧪 Potion Effect Events

### Jerarquía de eventos

```
EntityEvent
 └─ PotionEffectEvent (abstract)
     ├─ PotionEffectAddEvent        — efecto nuevo aplicado (Cancellable)
     │   └─ PotionEffectExtendEvent — efecto extendido/mejorado (hereda de Add)
     └─ PotionEffectRemoveEvent     — efecto eliminado (Cancellable)
         └─ PotionEffectExpireEvent — efecto expirado (setDuration > 0 = cancelado)
```

### `PotionEffectAddEvent`

Cancellable. Tiene `EffectAddReason`:

| Reason | Origen |
|---|---|
| `COMMAND` | `/effect` |
| `GOLDEN_APPLE` | Manzana dorada |
| `BEACON` | Beacon |
| `WITHER_SKULL` | Calavera de wither |
| `WITHER_SKELETON` | Esqueleto wither |
| `VILLAGER_CURE` | Cura de aldeano |
| `VILLAGER_LEVELUP` | Subida de nivel de aldeano |
| `SPIDER_POWERUP` | Arañas en difícil |
| `POTION_SPLASH` | Poción splash |
| `POTION_DRINK` | Poción bebible |
| `CUSTOM` | Plugin |
| `UNKNOWN` | Vanilla no cubierto |

### `PotionEffectExtendEvent` (extiende `PotionEffectAddEvent`)

```java
event.getOldEffect(); // estado del efecto antes de la mejora/extensión
```

### `PotionEffectExpireEvent` (extiende `PotionEffectRemoveEvent`)

```java
event.setDuration(600);  // > 0 extiende el efecto y cancela la expiración
event.setDuration(0);    // un-cancela, deja que expire
event.isCancelled();     // true si duration > 0
```

### Métodos nuevos en `LivingEntity`

```java
addPotionEffect(effect, reason);
addPotionEffect(effect, force, reason);
addPotionEffects(effects, reason);
```

```java
// Bloquear efectos de beacon en PvP
@EventHandler
public void onPotionAdd(PotionEffectAddEvent e) {
    if (e.getReason() == EffectAddReason.BEACON
            && e.getEffect().getType() == PotionEffectType.REGENERATION) {
        e.setCancelled(true);
    }
}

// Extender infinite un efecto que va a expirar
@EventHandler
public void onPotionExpire(PotionEffectExpireEvent e) {
    e.setDuration(Integer.MAX_VALUE);
}

// Aplicar efecto con motivo específico
player.addPotionEffect(
    new PotionEffect(PotionEffectType.SPEED, 200, 1),
    EffectAddReason.CUSTOM
);
```

---

## 📝 Title Compatibility Bridge

### `com.destroystokyo.paper.Title`

Puente de compatibilidad para plugins compilados contra la API original de Paper. Delega a `org.github.paperspigot.Title`.

```java
// Uso idéntico a la API de Paper
new Title("§6Título", "§7Subtítulo").send(player);
new Title("Hola", "Mundo", 10, 70, 20).send(player);
```

---

## �️ Entity Hide/Show API

### Métodos en `org.bukkit.entity.Player`

| Método | Qué hace |
|---|---|
| `hideEntity(Entity entity)` | Oculta **cualquier entidad** (mobs, items, etc.) del jugador |
| `showEntity(Entity entity)` | Vuelve a mostrar una entidad previamente ocultada |

> A diferencia de `hidePlayer`/`showPlayer` (solo para players), estos métodos funcionan con **cualquier `Entity`**.

```java
// Ocultar todos los mobs a un jugador
for (Entity entity : player.getWorld().getEntities()) {
    if (entity instanceof LivingEntity && !(entity instanceof Player)) {
        player.hideEntity(entity);
    }
}

// Mostrar de nuevo una entidad específica
player.showEntity(someEntity);
```

---

## �📦 Paquetes MASpigot

```
us.mineaqua.spigot
├── chunk/DummyChunkSnapshot.java          ← marker interface
├── knockback/KnockbackProfile.java        ← perfil KB
├── option/Option.java                     ← interfaz opción
├── option/SimpleOption.java               ← implementación concreta
└── server/                                ← interno (no API pública)
    ├── chunk/   (ChunkSectionSnapshot, ChunkSnapshotCommand)
    └── knockback/ (KnockbackManager, KnockbackCommand)
```

## 🔌 Dependencia Maven

```xml
<repository>
    <id>mineaqua-repo</id>
    <url>https://repo.mineaqua.net/private</url>
</repository>

<dependency>
    <groupId>com.hpfxd.pandaspigot</groupId>
    <artifactId>pandaspigot-api</artifactId>
    <version>1.8.8-R0.1-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

> Repositorio publicado vía `MINEAQUA_REPO_USERNAME` / `MINEAQUA_REPO_PASSWORD`.
