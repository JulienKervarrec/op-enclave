# Chapitre 1 -- Presentation de op-enclave

`op-enclave` est une modification relativement restreinte de l op-stack
qui prouve les transitions d etat dans une enclave AWS Nitro, puis
soumet les racines d etat resultantes a la chaine L1. Le gain
recherche est direct : cela supprime le besoin de la periode de
contestation de 7 jours du op-stack classique, et permet des retraits
immediats de L2 vers L1.

Dans l op-stack standard, un output L2 propose par le proposer doit
attendre 7 jours avant d etre considere final, le temps que quiconque
puisse le contester via un fault proof. `op-enclave` remplace cette
attente par une preuve materielle immediate : une enclave AWS Nitro (un
environnement d execution isole, dont le code exact est mesure et
atteste cryptographiquement par AWS) rejoue elle-meme la transition
d etat de maniere stateless, puis signe le nouvel output root avec une
cle privee generee et gardee a l interieur de l enclave. Le contrat
onchain n accepte un nouvel output que si sa signature provient d une
adresse dont la cle a ete generee par une enclave dont le code (mesure
via son PCR0) a ete explicitement approuve par l operateur de la chaine.

Le depot est organise en plusieurs composants qui collaborent : `contracts`
(les contrats Solidity onchain), `op-enclave` (la fonction de transition
stateless qui tourne dans l enclave Nitro elle-meme), `op-proposer`
(le service qui communique avec l enclave et soumet les propositions a
L1), `op-batcher` (une modification du batcher standard qui soumet les
batches immediatement des qu un retrait est detecte), `op-da` (un
service de disponibilite des donnees pour ecrire vers S3 ou un systeme
de fichiers), `op-withdrawer` (un utilitaire de retrait), et `tools`
(pour enregistrer les cles de signataire d enclave aupres du
`SystemConfigGlobal` et verifier les PCR0).
