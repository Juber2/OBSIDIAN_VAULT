## query para obtener usuarios, tipo y roles

esto es para saber si darle permiso de lectura o de lectura y escritura.

SELECT 
    UO.USER_ORGANIZATION_ID AS [Org. Id],
    O.DOCUMENT_TYPE_NUMBER AS RUC,
    O.COMERCIAL_NAME AS [Razón Social],
    U.DISPLAY_NAME AS Nombres,
    U.CONNECT_EMAIL AS [Connect Correo],
    U.EMAIL AS Correo,
    U.PHONE_NUMBER AS Celular,
    R.ROLE_NAME AS [Tipo de Rol],
    CASE 
        WHEN UO.SUPER_ADMIN = 1 THEN 'Super Admin'
        WHEN UO.IS_APPROVER = 1 THEN 'Aprovador'
    END AS [Tipo Cliente]
FROM COMMON.USER_ORGANIZATION UO
    INNER JOIN COMMON.ORGANIZATION O ON UO.ORGANIZATION_ID = O.ID
    INNER JOIN COMMON.USERS U ON UO.USER_ID = U.USER_ID
    INNER JOIN COMMON.ROLE R ON R.ROL_ID = UO.ROLE_ID
WHERE O.DOCUMENT_TYPE_NUMBER IN (
    '20603657862',
    '20199144961',
    '20538397971',
    '20390588551',
    '20502510470'
) ORDER BY O.DOCUMENT_TYPE_NUMBER DESC, UO.USER_ORGANIZATION_ID DESC;
-- -----------------------------------------------------------------------------------------------------

