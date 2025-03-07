# async/poll Ansible
### **Problème de base**  
Quand tu exécutes une commande dans Ansible, il attend que la commande soit **terminée** avant de passer à la suivante. Mais si la commande est **trop longue**, Ansible peut **perdre la connexion** ou **avoir un timeout**.  

Par exemple, si tu essaies de supprimer un grand nombre de fichiers avec cette commande :  

```yaml
- name: Supprimer les fichiers
  win_shell: |
    Remove-Item -Path 'C:\mon_dossier\*' -Force -Recurse
```

Si la suppression prend trop de temps, Ansible va **couper la connexion** et afficher une erreur.

---

### **Solution : Exécution Asynchrone**  
Les options `async` et `poll` permettent de **dire à Ansible d’exécuter la commande en arrière-plan**, sans attendre qu’elle soit finie immédiatement.  

#### **Explication des valeurs**
```yaml
- name: Supprimer les fichiers (sans timeout)
  win_shell: |
    Remove-Item -Path 'C:\mon_dossier\*' -Force -Recurse
  async: 120  # Donne jusqu'à 120 secondes à la commande pour s’exécuter
  poll: 10    # Vérifie toutes les 10 secondes si la commande est terminée
```

- `async: 120` → **Permet à Ansible de laisser tourner la commande jusqu'à 120 secondes** sans la bloquer.  
- `poll: 10` → **Ansible vérifie toutes les 10 secondes** si la commande est terminée.  

Si la commande finit **avant** 120 secondes, Ansible continue immédiatement. Sinon, il attend jusqu’à 120 secondes avant de considérer la tâche comme échouée.

---

### **Pourquoi c'est utile ?**
1. **Évite les timeouts** → Si la tâche est longue, Ansible ne perd pas la connexion.  
2. **Optimise l’exécution** → Ansible peut faire d’autres tâches pendant que celle-ci se termine.  
3. **Donne plus de temps aux commandes lentes** → Parfait pour des suppressions massives, des grosses compressions, etc.  

---

### **Cas sans async/poll (Erreur possible)**
```yaml
- name: Supprimer les fichiers
  win_shell: |
    Remove-Item -Path 'C:\mon_dossier\*' -Force -Recurse
```
🚨 **Si cette commande prend plus de 30 secondes, Ansible coupe la connexion.**  

### **Cas avec async/poll (Pas d’erreur)**
```yaml
- name: Supprimer les fichiers (sans timeout)
  win_shell: |
    Remove-Item -Path 'C:\mon_dossier\*' -Force -Recurse
  async: 120
  poll: 10
```
✅ **Ansible laisse la tâche tourner en arrière-plan et vérifie périodiquement si elle est terminée.**  

---
