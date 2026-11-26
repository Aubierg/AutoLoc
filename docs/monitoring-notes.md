# Notes de monitoring — AutoLoc

## Architecture

| Composant | Rôle |
|---|---|
| Spring Boot Actuator | Expose la santé et les métriques de l'application |
| Micrometer (registre Prometheus) | Met les métriques au format lisible par Prometheus |
| Prometheus | Interroge l'application toutes les 5 secondes et stocke les métriques |
| Grafana | Interroge Prometheus et affiche les graphiques |

Flux : l'application expose `/actuator/prometheus`, Prometheus la lit, Grafana affiche les données de Prometheus. Prometheus et Grafana tournent dans des conteneurs Docker, l'application tourne depuis IntelliJ sur le PC.

## Métriques choisies

| Panneau | Requête | Intérêt |
|---|---|---|
| JVM memory used | `jvm_memory_used_bytes` | Détecter une consommation mémoire anormale |
| CPU usage | `process_cpu_usage` | Suivre la charge processeur |
| HTTP request rate | `sum(rate(http_server_requests_seconds_count[1m]))` | Mesurer le trafic par seconde |
| HTTP response time p95 | `histogram_quantile(0.95, sum by (le) (rate(http_server_requests_seconds_bucket[5m])))` | Repérer les requêtes lentes |
| Live threads | `jvm_threads_live_threads` | Détecter une fuite de threads |

## Un problème résolu

Après l'installation de Docker Desktop, la commande `docker compose up -d` renvoyait « docker n'est pas reconnu » dans le terminal d'IntelliJ. Cause : IntelliJ avait été ouvert avant l'installation et n'avait pas la nouvelle variable PATH. Solution : fermer puis rouvrir IntelliJ (ou utiliser un PowerShell ouvert après l'installation).

## Sécurité

Seules les routes `/actuator/health` et `/actuator/prometheus` sont autorisées sans authentification dans Spring Security. Les détails de santé complets ne doivent pas être exposés publiquement en production.

## Création du tableau de bord

Le tableau de bord « AutoLoc — Application Monitoring » (5 panneaux) a été créé par import d'un fichier JSON dans Grafana, car l'éditeur de panneau n'était pas accessible dans la version utilisée.

## Arrêt

`docker compose down` arrête les conteneurs. L'application Spring Boot s'arrête séparément depuis IntelliJ.