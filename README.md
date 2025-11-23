# ShadowEco API Documentation

This document provides comprehensive details and usage examples for the `ShadowEcoAPI`, allowing developers to integrate their plugins seamlessly with the ShadowEco economy system.

The `ShadowEcoAPI` provides asynchronous methods to manage player balances, ensuring your plugin remains lag-free and performs optimally. All API methods return `CompletableFuture`s, which complete when the operation is finished.

---

## 🚀 Getting Started with the API

### 1. Project Setup

To use the `ShadowEcoAPI` in your plugin, you need to set up your project's `pom.xml` and `plugin.yml`.

#### `pom.xml` Dependency:

Your plugin must depend on the `ShadowEco` plugin at compile time. Add the following to your `pom.xml`:

```xml
<dependencies>
    <!-- Your existing dependencies -->

    <dependency>
        <groupId>com.shadoweco</groupId>
        <artifactId>ShadowEco</artifactId>
        <version>1.0.0</version> <!-- Use the version of the ShadowEco plugin you are targeting -->
        <scope>provided</scope>
    </dependency>
</dependencies>
```
**Note:** The `scope` is `provided` because the ShadowEco plugin itself will be present on the server at runtime. You do not need to bundle it with your plugin.

#### `plugin.yml` Soft-Depend:

To ensure the `ShadowEco` plugin loads before your plugin and its API is available, add a `softdepend` entry in your `plugin.yml`:

```yaml
name: YourPlugin
version: 1.0.0
main: com.yourplugin.YourPlugin
api-version: 1.19
softdepend: [ShadowEco]
```

### 2. Acquiring the API Instance

The recommended way to get an instance of the `ShadowEcoAPI` is through Bukkit's `ServicesManager`. You should typically do this in your plugin's `onEnable()` method.

```java
import com.shadoweco.economy.ShadowEcoAPI;
import org.bukkit.plugin.RegisteredServiceProvider;
import org.bukkit.plugin.java.JavaPlugin;
import java.util.logging.Logger;

public class YourPlugin extends JavaPlugin {

    private ShadowEcoAPI economyAPI;
    private static final Logger log = Logger.getLogger("Minecraft");

    @Override
    public void onEnable() {
        if (!setupEconomyAPI()) {
            log.severe("Disabled due to no ShadowEco API found!");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        log.info("Successfully hooked into ShadowEco API!");
        // Your plugin logic can now use 'economyAPI'
    }

    private boolean setupEconomyAPI() {
        if (getServer().getPluginManager().getPlugin("ShadowEco") == null) {
            return false; // ShadowEco is not installed
        }
        RegisteredServiceProvider<ShadowEcoAPI> rsp = getServer().getServicesManager().getRegistration(ShadowEcoAPI.class);
        if (rsp == null) {
            return false; // ShadowEco API not registered
        }
        economyAPI = rsp.getProvider();
        return economyAPI != null;
    }

    public ShadowEcoAPI getEconomyAPI() {
        return economyAPI;
    }
}
```

---

## 📚 API Methods

All methods in `ShadowEcoAPI` return a `CompletableFuture`. This means operations are non-blocking and will execute asynchronously. You should use `.thenAccept()`, `.thenApply()`, `.thenCompose()`, or `.exceptionally()` to handle the results. Avoid using `.get()` as it will block the main thread.

---

### `CompletableFuture<Double> getBalance(OfflinePlayer player)`

Retrieves the current balance of a specific player.

*   **Parameters:**
    *   `player`: The `org.bukkit.OfflinePlayer` whose balance you want to retrieve.
*   **Returns:**
    *   A `CompletableFuture` that will complete with the player's balance (as a `Double`).
*   **Example:**

    ```java
    // Assuming 'economyAPI' is initialized and 'targetPlayer' is an OfflinePlayer
    economyAPI.getBalance(targetPlayer).thenAccept(balance -> {
        // This code runs asynchronously after the balance is fetched
        Bukkit.getScheduler().runTask(myPlugin, () -> { // Always run Bukkit API calls on main thread
            player.sendMessage(targetPlayer.getName() + "'s balance: " + balance);
        });
    }).exceptionally(ex -> {
        myPlugin.getLogger().severe("Error getting balance for " + targetPlayer.getName() + ": " + ex.getMessage());
        return null;
    });
    ```

---

### `CompletableFuture<Boolean> deposit(OfflinePlayer player, double amount)`

Deposits a specified amount into a player's account.

*   **Parameters:**
    *   `player`: The `org.bukkit.OfflinePlayer` to deposit money to.
    *   `amount`: The `double` amount to deposit. Must be positive.
*   **Returns:**
    *   A `CompletableFuture` that will complete with `true` if the deposit was successful. If `amount` is zero or negative, it will complete with `false`.
*   **Example:**

    ```java
    // Assuming 'economyAPI' is initialized and 'targetPlayer' is an OfflinePlayer
    double depositAmount = 100.00;
    economyAPI.deposit(targetPlayer, depositAmount).thenAccept(success -> {
        Bukkit.getScheduler().runTask(myPlugin, () -> {
            if (success) {
                player.sendMessage("Successfully deposited " + depositAmount + " to " + targetPlayer.getName());
            } else {
                player.sendMessage("Failed to deposit " + depositAmount + " to " + targetPlayer.getName() + ". (Amount must be positive)");
            }
        });
    }).exceptionally(ex -> {
        myPlugin.getLogger().severe("Error depositing to " + targetPlayer.getName() + ": " + ex.getMessage());
        return null;
    });
    ```

---

### `CompletableFuture<Boolean> withdraw(OfflinePlayer player, double amount)`

Withdraws a specified amount from a player's account.

*   **Parameters:**
    *   `player`: The `org.bukkit.OfflinePlayer` to withdraw money from.
    *   `amount`: The `double` amount to withdraw. Must be positive.
*   **Returns:**
    *   A `CompletableFuture` that will complete with `true` if the withdrawal was successful (player had sufficient funds).
    *   Completes with `false` if the player had insufficient funds or if `amount` is zero/negative.
*   **Example:**

    ```java
    // Assuming 'economyAPI' is initialized and 'targetPlayer' is an OfflinePlayer
    double withdrawAmount = 50.00;
    economyAPI.withdraw(targetPlayer, withdrawAmount).thenAccept(success -> {
        Bukkit.getScheduler().runTask(myPlugin, () -> {
            if (success) {
                player.sendMessage("Successfully withdrew " + withdrawAmount + " from " + targetPlayer.getName());
            } else {
                player.sendMessage("Failed to withdraw " + withdrawAmount + " from " + targetPlayer.getName() + ". (Insufficient funds or invalid amount)");
            }
        });
    }).exceptionally(ex -> {
        myPlugin.getLogger().severe("Error withdrawing from " + targetPlayer.getName() + ": " + ex.getMessage());
        return null;
    });
    ```

---

### `CompletableFuture<Boolean> setBalance(OfflinePlayer player, double amount)`

Sets a player's balance to a specific amount.

*   **Parameters:**
    *   `player`: The `org.bukkit.OfflinePlayer` whose balance is to be set.
    *   `amount`: The `double` amount to set as the new balance. Must be non-negative.
*   **Returns:**
    *   A `CompletableFuture` that will complete with `true` if the balance was set successfully.
    *   Completes with `false` if `amount` is negative.
*   **Example:**

    ```java
    // Assuming 'economyAPI' is initialized and 'targetPlayer' is an OfflinePlayer
    double newBalance = 1000.00;
    economyAPI.setBalance(targetPlayer, newBalance).thenAccept(success -> {
        Bukkit.getScheduler().runTask(myPlugin, () -> {
            if (success) {
                player.sendMessage(targetPlayer.getName() + "'s balance set to " + newBalance);
            } else {
                player.sendMessage("Failed to set " + targetPlayer.getName() + "'s balance to " + newBalance + ". (Amount must be non-negative)");
            }
        });
    }).exceptionally(ex -> {
        myPlugin.getLogger().severe("Error setting balance for " + targetPlayer.getName() + ": " + ex.getMessage());
        return null;
    });
    ```

---

### `CompletableFuture<java.util.Map<java.util.UUID, Double>> getTopBalances(int count)`

Retrieves a map of the top player balances.

*   **Parameters:**
    *   `count`: An `int` representing the number of top players to retrieve.
*   **Returns:**
    *   A `CompletableFuture` that will complete with a `Map` where keys are player `UUID`s and values are their balances. The map is sorted in descending order of balance.
*   **Example:**

    ```java
    // Assuming 'economyAPI' is initialized
    int topN = 5;
    economyAPI.getTopBalances(topN).thenAccept(topBalances -> {
        Bukkit.getScheduler().runTask(myPlugin, () -> {
            player.sendMessage("--- Top " + topN + " Players ---");
            int rank = 1;
            for (Map.Entry<UUID, Double> entry : topBalances.entrySet()) {
                OfflinePlayer topPlayer = Bukkit.getOfflinePlayer(entry.getKey());
                String playerName = topPlayer.getName() != null ? topPlayer.getName() : "Unknown";
                player.sendMessage(rank + ". " + playerName + ": " + entry.getValue());
                rank++;
            }
        });
    }).exceptionally(ex -> {
        myPlugin.getLogger().severe("Error getting top balances: " + ex.getMessage());
        return null;
    });
    ```

---

## 📄 License
This API is open-sourced under the MIT License. See the `LICENSE` file for more details.
