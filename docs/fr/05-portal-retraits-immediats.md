# Chapitre 5 -- Portal.sol : des retraits sans periode de contestation

`contracts/src/Portal.sol` est, comme `OutputOracle`, une adaptation du
`OptimismPortal` standard de l op-stack. Le coeur du contrat -- la
preuve d un retrait via une inclusion Merkle dans
`proveAndFinalizeWithdrawalTransaction` -- reste conceptuellement
identique a l original : le contrat lit l output root associe a un
index donne, verifie qu il correspond bien a la structure de preuve
fournie (`Hashing.hashOutputRootProof`), calcule l emplacement de
stockage ou le hash du retrait aurait du etre ecrit dans le contrat
`L2ToL1MessagePasser`, puis verifie une preuve d inclusion Merkle
(`SecureMerkleTrie.verifyInclusionProof`) contre le root de stockage
contenu dans l output. Le nom de la fonction le dit explicitement :
`proveAndFinalizeWithdrawalTransaction` prouve ET finalise en une seule
transaction -- contrairement a l op-stack standard qui separe les deux
etapes par la periode de contestation de 7 jours, ici la preuve suffit
immediatement, parce que l output root lui-meme n a pu etre publie
que s il portait la signature d une enclave de confiance (chapitres 2 a 4).

La fonction `finalizeWithdrawalTransaction` d origine, elle, est reduite
a un simple commentaire `// do nothing` -- elle existe encore dans
l interface pour compatibilite mais n a plus de role, puisque
`proveAndFinalizeWithdrawalTransaction` fait tout en une fois.
`isOutputFinalized` conserve la notion de periode de finalisation
(`_isFinalizationPeriodElapsed`, qui compare simplement le timestamp
au bloc courant), mais elle n est plus un prealable a la finalisation
du retrait lui-meme dans ce contrat -- elle reste disponible comme
utilitaire de lecture pour la compatibilite avec l ecosysteme op-stack
existant.

Le reste du contrat (depots ETH et ERC20, `depositTransaction`,
`depositERC20Transaction`, mesure de ressources via `ResourceMetering`,
support de token de gas personnalise) est repris presque a l identique
de l `OptimismPortal` standard et n est pas specifique a op-enclave.
