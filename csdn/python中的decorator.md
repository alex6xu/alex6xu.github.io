def build(user, build_name): if not is_user_valid(user):
redirect("/auth/login/") return False #do something here return
create_building(build_name)

