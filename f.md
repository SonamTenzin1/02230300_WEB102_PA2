const app = new Hono()

app.get('/', (c) => {
  return c.text('Hello Hono!')
})

const port = 3000
console.log(`app is running on port ${port}`)

serve({
  fetch: app.fetch,
  port
})


type CustomVariables = JwtVariables

const app = new Hono<{ Variables: CustomVariables }>();
const prisma = new PrismaClient();

app.use("/*", cors());

app.use(
  "/secure/*",
  jwt({
    secret: 'secretKey',
  })
);

// Authentication Endpoints
app.post("/signup", async (context) => {
  const requestBody = await context.req.json();
  const userEmail = requestBody.email;
  const userPassword = requestBody.password;

  const hashedPassword = await Bun.password.hash(userPassword, {
    algorithm: "bcrypt",
    cost: 4,
  });

  try {
    const newUser = await prisma.user.create({
      data: {
        email: userEmail,
        hashedPassword: hashedPassword,
      },
    });

    return context.json({ message: `${newUser.email} created successfully` });
  } catch (error) {
    if (error instanceof Prisma.PrismaClientKnownRequestError) {
      if (error.code === "P2002") {
        return context.json({ message: "Email already exists" });
      }
    }
    throw new HTTPException(500, { message: "Internal app Error" });
  }
});

app.post("/signin", async (context) => {
  const requestBody = await context.req.json();
  const userEmail = requestBody.email;
  const userPassword = requestBody.password;

  const existingUser = await prisma.user.findUnique({
    where: { email: userEmail },
    select: { id: true, hashedPassword: true },
  });

  if (!existingUser) {
    return context.json({ message: "User not found" }, 404);
  }

  const passwordMatch = await Bun.password.verify(
    userPassword,
    existingUser.hashedPassword,
    "bcrypt"
  );

  if (passwordMatch) {
    const tokenPayload = {
      sub: existingUser.id,
      exp: Math.floor(Date.now() / 1000) + 60 * 60, // Token expires in 60 minutes
    };
    const tokenSecret = "secretKey";
    const authToken = sign(tokenPayload, tokenSecret);
    return context.json({ message: "Login successful", token: authToken });
  } else {
    throw new HTTPException(401, { message: "Invalid credentials" });
  }
});

// Fetch Pokemon Data from PokeAPI
app.get("/pokemon/:name", async (context) => {
  const { name } = context.req.param();

  try {
    const apiResponse = await axios.get(`https://pokeapi.co/api/v2/pokemon/${name}`);
    return context.json({ data: apiResponse.data });
  } catch (error) {
    return context.json({ message: "Pokemon not found" }, 404);
  }
});

// Protected User Resource Endpoints
app.post("/secure/catch", async (context) => {
  const jwtPayload = context.get('jwtPayload');
  if (!jwtPayload) {
    throw new HTTPException(401, { message: "Unauthorized" });
  }
  
  const requestBody = await context.req.json();
  const pokemonName = requestBody.name;

  let pokemon = await prisma.pokemon.findUnique({ where: { name: pokemonName } });
  
  if (!pokemon) {
    pokemon = await prisma.pokemon.create({
      data: { name: pokemonName }
    });
  }

  const caughtPokemonRecord = await prisma.caughtPokemon.create({
    data: {
      userId: jwtPayload.sub,
      pokemonId: pokemon.id
    }
  });

  return context.json({ message: "Pokemon caught", data: caughtPokemonRecord });
});

app.delete("/secure/release/:id", async (context) => {
  const jwtPayload = context.get('jwtPayload');
  if (!jwtPayload) {
    throw new HTTPException(401, { message: "Unauthorized" });
  }

  const { id } = context.req.param();

  await prisma.caughtPokemon.deleteMany({
    where: { id: id, userId: jwtPayload.sub }
  });

  return context.json({ message: "Pokemon released" });
});

app.get("/secure/caught", async (context) => {
  const jwtPayload = context.get('jwtPayload');
  if (!jwtPayload) {
    throw new HTTPException(401, { message: "Unauthorized" });
  }

  const caughtPokemonList = await prisma.caughtPokemon.findMany({
    where: { userId: jwtPayload.sub },
    include: { pokemon: true }
  });

  return context.json({ data: caughtPokemonList });
});

export default app;
