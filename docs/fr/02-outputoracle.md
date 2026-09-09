# Chapitre 2 -- OutputOracle : la chaine de confiance onchain

`contracts/src/OutputOracle.sol` est une copie modifiee du
`L2OutputOracle` standard de l op-stack, avec plusieurs differences
assumees et documentees en commentaire : un nombre limite d outputs
stockes (`maxOutputCount`, en buffer circulaire), la suppression de la
periode de finalisation classique, une verification de signature
supplementaire pour `proposeL2Output`, et la possibilite de proposer un
output a tout moment (pas seulement a intervalle regulier).

`proposeL2Output` verifie d abord que l appelant est bien le `proposer`
enregistre et que le numero de bloc L2 avance strictement. Si
`proofsEnabled` est actif (`enableProofs()` est une fonction a sens
unique, irreversible, appelable uniquement par le proposer), la fonction
exige beaucoup plus : elle recupere le hash du bloc L1 donne
(`blockhash`, qui n est disponible que pour les 256 derniers blocs,
ancrant donc la preuve a un contexte L1 recent), puis reconstruit un
message `keccak256(configHash, blockHash, l2BlockNumber, previousOutputRoot,
outputRoot)` et recupere le signataire via `ECDSA.recover`. Ce
signataire doit figurer dans `systemConfigGlobal.validSigners` -- c est
la seule porte d entree : un output n est accepte que s il est signe
par une cle dont l enclave a ete prealablement approuvee (chapitre 3).

Le buffer d outputs est circulaire : une fois `maxOutputCount` atteint,
les nouveaux outputs ecrasent les plus anciens
(`l2Outputs[latestOutputIndex] = op`). Le commentaire du code previent
explicitement que si un retrait est ancien, l output qui le couvrait
peut avoir ete ecrase par une proposition plus recente -- une consequence
directe du fait que ce contrat, contrairement a l original, ne garde
pas un historique complet.
