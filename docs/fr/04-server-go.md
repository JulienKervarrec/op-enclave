# Chapitre 4 -- Le serveur d enclave (Go) : cles, attestations et signature des propositions

`op-enclave/op-enclave/enclave/server.go` est le code qui tourne a
l interieur de l enclave Nitro elle-meme. A l initialisation
(`NewServer`), il tente d ouvrir une session vers le Nitro Secure
Module (NSM, le peripherique materiel qui fournit l entropie et les
attestations) ; s il n y arrive pas, il bascule en "mode local" avec
`rand.Reader` standard -- un mode qui n autorise une cle de signataire
fixee par variable d environnement que pour le developpement, jamais en
production reelle. En mode NSM reel, il recupere le PCR0 courant
(`DescribePCR`) et genere deux paires de cles a l interieur de
l enclave : une cle de dechiffrement RSA-4096 et une cle de signature
ECDSA secp256k1 -- toutes deux generees avec l entropie fournie par le
NSM, jamais exportees en clair.

`publicKeyAttestation` demande au NSM un document d attestation qui lie
une cle publique donnee a la mesure PCR0 courante de l enclave -- c est
ce document que `SystemConfigGlobal.registerSigner` (chapitre 3)
verifiera onchain. Un mecanisme plus subtil gere le cas ou plusieurs
instances d enclave doivent partager la meme cle de signature (par
exemple lors d un redemarrage ou d une bascule) :
`EncryptedSignerKey` verifie l attestation d une AUTRE enclave (via la
bibliotheque `nitrite`, en verifiant la chaine jusqu aux racines AWS
codees en dur dans `DefaultCARoots`), s assure que son PCR0 correspond
exactement au sien propre, puis chiffre sa propre cle de signature avec
la cle publique RSA de cette autre enclave et la lui transmet ;
`SetSignerKey`, cote receveur, la dechiffre avec sa cle RSA privee et
l adopte. Ce transfert ne fonctionne qu entre deux enclaves executant
exactement le meme code (meme PCR0) -- une nouvelle version du binaire
ne peut jamais recuperer la cle d une ancienne sans nouvelle
approbation `registerPCR0`.

`ExecuteStateless` est le coeur metier : elle recoit un temoin
d execution (`ExecutionWitness`, contenant l etat necessaire pour
rejouer un bloc sans acces a une base de donnees complete), rejoue la
transition d etat via `ExecuteStateless` (la fonction de transition
op-stack, stateless), calcule l output root avant et apres
(`OutputRootV0`, qui hache le root d etat, le root de stockage du
message passer, et le hash du header), puis signe le message
`keccak256(configHash, l1OriginHash, l2BlockNumber, prevOutputRoot,
outputRoot)` avec la cle ECDSA de l enclave -- exactement le message
que `OutputOracle.proposeL2Output` recalculera et verifiera onchain.
`Aggregate` permet de chainer plusieurs propositions individuelles en
une seule, en reverifiant chaque signature intermediaire avant de
signer le resultat agrege.
