# Loot test sets

Those are test sets for loot generation in Minecraft.
For every version, all the loot table are generated for a
random set of seeds. We provide also variation of such loot
sets with different luck.

## Test content

Test are organized in branch, you can checkout each branch with
format `v<version>`, you can also checkout main to get a sample of
tests.
Five different types of sets are provided:
- Small test set, it contains 1_000 random seeds.
- Small test set with luck, it contains 10 seeds with 200 variations of lucks for each.
- Large test set, it contains 100_000 random seeds, the small test set is included.
- Large test set with luck, it contains 100 random seeds with 2000 variation of
  lucks for each, this makes it a 2_000_000 test set, it contains the small test set fully.
- A fully random set to validate everything, contains 5_000_000 seeds and luck variation,
  those are completly independant from the previous sets, this can be used to validate everything.



## Supported versions


* 1.14


## Example of programs used

### 1.14

```java
Bootstrap.register();
JsonArray root = new JsonArray();

int max = 100;
float maxLuck = 100.0F;
boolean doLuck = true;

boolean pretty = max < 2000 ? true : false;


for (int i = 0; i < max; i++) {
    if ((i % (max / 100)) == 0) {
        System.out.println("Progress " + i / (max / 100.0));
    }
    for (float luck = -maxLuck; luck < maxLuck; luck += 0.1F) {
        final float currentLuck = luck;
        long seed = new Random(i).nextLong();
        Supplier<LootContext> lootContext = () -> {
            LootContext.Builder builder = new LootContext.Builder(null).withSeed(seed);
            if (doLuck) {
                builder = builder.withLuck(currentLuck);
            }
            return builder.build(new LootParameterSet.Builder().build());
        };
        JsonObject current = new JsonObject();
        current.addProperty("seed", seed);
        if (doLuck) {
            current.addProperty("luck", luck);
        }
        JsonObject array = new JsonObject();
        Function<ItemStack, JsonElement> stackToJSON = loot -> {
            JsonObject lootJson = new JsonObject();
            lootJson.addProperty("count", loot.getCount());
            lootJson.addProperty("item", loot.getItem().toString());
            JsonArray enchantments = new JsonArray();
            loot.getEnchantmentTagList().forEach(x -> {
                JsonObject enchJson = new JsonObject();
                enchJson.addProperty("id", ((CompoundNBT) x).get("id").getString());
                enchJson.addProperty("lvl", ((ShortNBT) ((CompoundNBT) x).get("lvl")).getInt());
                enchantments.add(enchJson);
            });
            lootJson.add("enchantments", enchantments);
            return lootJson;
        };
        new ChestLootTables().accept((x, y) -> {
            JsonArray loots = new JsonArray();
            y.build().generate(lootContext.get()).stream().map(stackToJSON).forEach(loots::add);
            array.add(x.getPath(), loots);
        });
        new FishingLootTables().accept((x, y) -> {
            JsonArray loots = new JsonArray();
            y.build().generate(lootContext.get()).stream().map(stackToJSON).forEach(loots::add);
            array.add(x.getPath(), loots);
        });
        current.add("loots", array);
        root.add(current);
        if (!doLuck) {
            break;
        }
    }
}

Gson gson = (pretty ? new GsonBuilder().setPrettyPrinting() : new GsonBuilder()).create();
FileWriter file = new FileWriter("./1.14.json");
gson.toJson(root, file);
file.close();
```