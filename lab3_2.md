def create_person(name: str, born_in: dt.datetime) -> Dict[str, Any]:
    return {
        "id": id(None) + hash(name) + hash(str(born_in)),
        "name": name,
        "born_in": born_in,
        "friends": []
    }

def person_add_friend(person: Dict[str, Any], friend: Dict[str, Any]) -> None:
    person["friends"].append(friend)
    friend["friends"].append(person)

def encode_person_fp(person: Dict[str, Any]) -> bytes:
    visited: Set[int] = set()
    def person_to_dict(p: Dict[str, Any]) -> Dict[str, Any]:
        person_id = p["id"]
        if person_id in visited:
            return {"_ref": person_id}
        visited.add(person_id)
        return {
            "id": person_id,
            "name": p["name"],
            "born_in": p["born_in"].isoformat(),
            "friends": [person_to_dict(f) for f in p["friends"]]
        }
    data = person_to_dict(person)
    return json.dumps(data).encode('utf-8')

def decode_person_fp(data: bytes) -> Dict[str, Any]:
    obj_map: Dict[int, Dict[str, Any]] = {}
    def dict_to_person(d: Dict[str, Any]) -> Dict[str, Any]:
        if "_ref" in d:
            return obj_map[d["_ref"]]
        person_id = d["id"]
        if person_id in obj_map:
            return obj_map[person_id]
        person: Dict[str, Any] = {
            "id": person_id,
            "name": d["name"],
            "born_in": dt.datetime.fromisoformat(d["born_in"]),
            "friends": []
        }
        obj_map[person_id] = person
        person["friends"] = [dict_to_person(f) for f in d["friends"]]
        return person
    parsed = json.loads(data.decode('utf-8'))
    return dict_to_person(parsed)