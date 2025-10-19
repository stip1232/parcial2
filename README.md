# parcial2# Sistema simple de gestión de actividades (nivel novato)
# — Corto, con variables sencillas y funciones pequeñas (modular).
# — Usa listas, tuplas y diccionarios.
# — Menús: Usuarios (CRUD) y Tareas (CRUD) + reportes.

# ===== Catálogos (tuplas: datos fijos) =====
estados = ("pendiente", "en_progreso", "completada")
roles = ("Admin", "Lider", "Colab")

# ===== Almacenamiento en memoria =====
usuarios = {}   # dict: "U1" -> {nombre, ident, contacto:{tel,email}, rol, tareas:[]}
tareas = []     # lista de dicts: {id, titulo, desc, estado, uid, meta:{prioridad,tags}}

# Contadores para IDs (muy simple)
uid_n = 0
tid_n = 0

# ===== Utilidades =====
def pedir(txt):
    v = input(txt).strip()
    while not v:
        print("No puede estar vacío.")
        v = input(txt).strip()
    return v


def elegir(txt, opciones):
    print(txt, opciones)
    v = input("> ").strip()
    while v not in opciones:
        print("Opción inválida.")
        v = input("> ").strip()
    return v


# ===== Usuarios (CRUD) =====
def u_crear():
    global uid_n
    n = pedir("Nombre: ")
    i = pedir("Identificación: ")
    t = pedir("Teléfono: ")
    e = pedir("Email: ")
    r = elegir("Rol:", roles)
    uid_n += 1
    uid = f"U{uid_n}"
    usuarios[uid] = {
        "nombre": n,
        "ident": i,
        "contacto": {"tel": t, "email": e},   # dict anidado
        "rol": r,
        "tareas": []                              # lista
    }
    print("Usuario creado:", uid)


def u_listar():
    if not usuarios:
        print("Sin usuarios"); return
    for uid, u in usuarios.items():
        print(uid, "|", u["nombre"], "|", u["rol"], "| tareas:", len(u["tareas"]))


def u_ver():
    uid = pedir("ID usuario (p.e. U1): ")
    u = usuarios.get(uid)
    if not u:
        print("No existe"); return
    print(uid, u)


def u_actualizar():
    uid = pedir("ID usuario: ")
    u = usuarios.get(uid)
    if not u:
        print("No existe"); return
    n = input(f"Nombre [{u['nombre']}]: ").strip() or u['nombre']
    i = input(f"Identificación [{u['ident']}]: ").strip() or u['ident']
    t = input(f"Tel [{u['contacto']['tel']}]: ").strip() or u['contacto']['tel']
    e = input(f"Email [{u['contacto']['email']}]: ").strip() or u['contacto']['email']
    r = input(f"Rol {roles} [{u['rol']}]: ").strip() or u['rol']
    if r not in roles:
        print("Rol inválido, se deja igual."); r = u['rol']
    u.update({"nombre": n, "ident": i, "rol": r, "contacto": {"tel": t, "email": e}})
    print("Actualizado")


def u_eliminar():
    uid = pedir("ID usuario: ")
    u = usuarios.get(uid)
    if not u:
        print("No existe"); return
    if u["tareas"]:
        print("No se puede: tiene tareas. Bórralas o reasígnalas primero.")
        return
    del usuarios[uid]
    print("Usuario eliminado")


# ===== Tareas (CRUD) =====
def t_crear():
    global tid_n
    ti = pedir("Título: ")
    de = pedir("Descripción: ")
    es = elegir("Estado:", estados)
    uid = pedir("Asignar a (ID usuario): ")
    if uid not in usuarios:
        print("No se puede: el usuario no existe"); return
    pr = elegir("Prioridad:", ("baja", "media", "alta"))
    tg = [x.strip() for x in input("Tags (coma, opcional): ").split(",") if x.strip()]
    tid_n += 1
    tid = f"T{tid_n}"
    t = {"id": tid, "titulo": ti, "desc": de, "estado": es, "uid": uid,
         "meta": {"prioridad": pr, "tags": tg}}
    tareas.append(t)
    usuarios[uid]["tareas"].append(tid)
    print("Tarea creada:", tid)


def t_listar(es_filtro=None):
    lista = tareas
    if es_filtro:
        if es_filtro not in estados:
            print("Estado inválido"); return
        lista = [x for x in tareas if x["estado"] == es_filtro]
    if not lista:
        print("Sin tareas"); return
    for x in lista:
        nom = usuarios.get(x["uid"], {}).get("nombre", "?")
        print(x["id"], "|", x["titulo"], "|", x["estado"], "|", x["uid"], nom)


def t_ver():
    tid = pedir("ID tarea (p.e. T1): ")
    x = next((y for y in tareas if y["id"] == tid), None)
    if not x:
        print("No existe"); return
    print(x)


def t_actualizar():
    tid = pedir("ID tarea: ")
    i = next((k for k, y in enumerate(tareas) if y["id"] == tid), None)
    if i is None:
        print("No existe"); return
    x = tareas[i]
    ti = input(f"Título [{x['titulo']}]: ").strip() or x['titulo']
    de = input(f"Desc [{x['desc']}]: ").strip() or x['desc']
    es = input(f"Estado {estados} [{x['estado']}]: ").strip() or x['estado']
    if es not in estados:
        print("Estado inv., se deja igual"); es = x['estado']
    nu = input(f"Asignado a (ID) [{x['uid']}]: ").strip() or x['uid']
    if nu not in usuarios:
        print("Usuario inv., se deja igual"); nu = x['uid']
    pr = input(f"Prioridad (baja/media/alta) [{x['meta']['prioridad']}]: ").strip() or x['meta']['prioridad']
    if pr not in ("baja","media","alta"):
        print("Prio inv., se deja igual"); pr = x['meta']['prioridad']
    tg_in = input(f"Tags (coma) [{', '.join(x['meta']['tags']) if x['meta']['tags'] else ''}]: ")
    tg = [z.strip() for z in tg_in.split(",") if z.strip()] if tg_in else x['meta']['tags']

    if nu != x['uid']:
        if x['id'] in usuarios[x['uid']]["tareas"]:
            usuarios[x['uid']]["tareas"].remove(x['id'])
        usuarios[nu]["tareas"].append(x['id'])

    x.update({"titulo": ti, "desc": de, "estado": es, "uid": nu})
    x['meta'].update({"prioridad": pr, "tags": tg})
    print("Tarea actualizada")


def t_eliminar():
    tid = pedir("ID tarea: ")
    i = next((k for k, y in enumerate(tareas) if y["id"] == tid), None)
    if i is None:
        print("No existe"); return
    u = tareas[i]["uid"]
    if tid in usuarios.get(u, {}).get("tareas", []):
        usuarios[u]["tareas"].remove(tid)
    tareas.pop(i)
    print("Tarea eliminada")


# ===== Reportes =====
def r_por_usuario():
    uid = pedir("ID usuario: ")
    if uid not in usuarios:
        print("No existe"); return
    ids = usuarios[uid]["tareas"]
    if not ids:
        print("(sin tareas)"); return
    for x in (y for y in tareas if y["id"] in ids):
        print(x["id"], x["titulo"], x["estado"], x["meta"]["prioridad"])


def r_total_por_estado():
    c = {e: 0 for e in estados}
    for x in tareas:
        c[x["estado"]] += 1
    print(c)


# ===== Menús =====
def menu_usuarios():
    while True:
        print("\n[Usuarios] 1-Crear 2-Listar 3-Ver 4-Actualizar 5-Eliminar 0-Volver")
        op = input("> ").strip()
        if op == "1": u_crear()
        elif op == "2": u_listar()
        elif op == "3": u_ver()
        elif op == "4": u_actualizar()
        elif op == "5": u_eliminar()
        elif op == "0": break
        else: print("Inválido")


def menu_tareas():
    while True:
        print("\n[Tareas] 1-Crear 2-Listar 3-Listar x estado 4-Ver 5-Actualizar 6-Eliminar 7-Rep usuario 8-Rep estados 0-Volver")
        op = input("> ").strip()
        if op == "1": t_crear()
        elif op == "2": t_listar()
        elif op == "3": t_listar(elegir("Estado:", estados))
        elif op == "4": t_ver()
        elif op == "5": t_actualizar()
        elif op == "6": t_eliminar()
        elif op == "7": r_por_usuario()
        elif op == "8": r_total_por_estado()
        elif op == "0": break
        else: print("Inválido")


def menu():
    while True:
        print("\n== Sistema == 1-Usuarios 2-Tareas 0-Salir")
        op = input("> ").strip()
        if op == "1": menu_usuarios()
        elif op == "2": menu_tareas()
        elif op == "0": print("Adiós"); break
        else: print("Inválido")


if __name__ == "__main__":
    menu()
