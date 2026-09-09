# Parcours francais : op-enclave (transition d etat op-stack dans une enclave AWS Nitro)

Lecture commentee du depot base/op-enclave : une modification de l op-stack qui remplace la periode de contestation de 7 jours par une preuve materielle produite dans une enclave AWS Nitro, permettant des retraits immediats de L2 vers L1.

Sommaire :

Chapitre 1 Presentation de op-enclave. Chapitre 2 OutputOracle, la chaine de confiance onchain. Chapitre 3 SystemConfigGlobal, enregistrer une enclave comme signataire de confiance. Chapitre 4 Le serveur d enclave (Go), cles, attestations et signature des propositions. Chapitre 5 Portal.sol, des retraits sans periode de contestation. Chapitre 6 Limites et perimetre de ce parcours.

Ce parcours est une lecture pedagogique du code source et de la documentation du depot, sans installation ni execution du projet.
