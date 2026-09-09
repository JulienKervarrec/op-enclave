# Chapitre 6 -- Limites et perimetre de ce parcours

Ce parcours couvre le mecanisme central de op-enclave : la chaine de
confiance complete, depuis l attestation materielle AWS Nitro jusqu a
la finalisation immediate des retraits onchain. Cote contrats :
`OutputOracle.sol` (verification de signature conditionnee a
`proofsEnabled`) et `SystemConfigGlobal.sol` (enregistrement de PCR0 et
de signataires attestes). Cote enclave : `server.go` (generation de
cles, attestation, transfert de cle chiffre entre enclaves, signature
des propositions dans `ExecuteStateless`/`Aggregate`). Cote retraits :
`Portal.sol` et la fusion preuve+finalisation en une seule transaction.

Sont volontairement laisses hors champ : le detail de la fonction de
transition d etat elle-meme (le paquet qui implemente
`ExecuteStateless` au sens strict, heritee et adaptee du client Geth
sans-etat, qui est un sujet a part entiere) ; les autres contrats du
dossier `contracts/src/` non essentiels au mecanisme de confiance
(`DeployChain.sol`, `OwnableConfig.sol`, `OwnableManagedUpgradeable.sol`,
`OwnerConfig.sol`, `ResolvingProxy.sol`, `ResolvingProxyFactory.sol`,
`SystemConfigOwnable.sol`) ; les services `op-batcher`, `op-da` et
`op-withdrawer`, mentionnes dans le README pour leur role mais non
detailles ici ; les outils de `tools/` pour l enregistrement pratique
des cles et la verification des PCR0 en ligne de commande ; le
deploiement Docker complet documente dans le README (`make
deploy-cert-manager`, `make deploy`, `make testnet`) ; et le contenu
des rapports d audit du dossier `audits/`.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment une preuve materielle
(l attestation AWS Nitro) peut remplacer une preuve economique
(la periode de contestation de 7 jours) dans le modele de securite d un
rollup, sans pretendre couvrir l integralite de l implementation de la
fonction de transition d etat sans-etat elle-meme.
