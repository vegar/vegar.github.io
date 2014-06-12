---
title: '1st particle-challenge: Getting something unto the screen'
author: Vegar
layout: post
permalink: /2009/1st-particle-challenge-getting-something-unto-the-screen/
categories:
  - Delphi
  - Development
tags:
  - Challenge
  - OpenGL
  - Particle
---
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2009/07/something_thumb.png" alt="No firework, but its something" title="No firework, but it's something" />A couple of days ago, I took a <a href="http://blog.vi-kan.net/2009/the-“particle-engines-is-not-that-hard’-challenge/" title="The &quot;Particle-engines-is-not-that-hard'-challenge">challenge</a> to show that making a particle engine is not that hard. I also stated that the first part of the challenge would be to get something unto the screen, and that is as far as I have come. I have enough code to emit particles unto the screen, beautifully rendered as small triangles in various colors.</p>

<p>Well – its no firework, but it&#8217;s a start…</p>

<p>For those who are interested in the code, it&#8217;s available for  <a href="http://svn.vi-kan.net/particle/0.1" title="The code for particle challenge">download here</a>. The lib-folder contains two libraries that I use. First of all, there is the <a href="http://www.delphigl.com">Free Pascal OpenGL Headers</a> used for rendering. Second, there is a folder called &#8216;SDL&#8217;, which is a older version of the <a href="http://sourceforge.net/projects/decal/">Delphi Container and Algorithm Library</a> (DeCAL).  I started using it for many, many years ago, back when it was a commercial product. It was called Standard Delphi Library, but was renamed to avoid confusion with &#8216;<a href="http://www.libssld.org" title="Simple DirectMedia Library">Simple DirectMedia Library</a>&#8217;. I guess it&#8217;s time to find something else, and when I start using Delphi 2009, I will for sure.</p>

<p>So, how did I put something unto the screen?</p>

<p>I started off with a simple TParticle class with information about position, velocity, color, time to live and age.</p>

<pre><code> TParticle = class(TObject)
  public
    constructor Create;

    procedure Update(DeltaFrame: int64);

    property Position: TVector3D read FPosition write FPosition;
    property Velocity: TVector3D read FVelocity write FVelocity;

    property Color: TColorFA read FColor write FColor;

    property TimeToLive: int64 read FTimeToLive write FTimeToLive;
    property Age: int64 read GetAge;
    property Dead: boolean read FDead write FDead;
  end;
</code></pre>

<p>Then followed a TParticleManager to hold all the particles.</p>

<pre><code> TParticleManager = class(TObject)
  public
    constructor Create;
    destructor Destroy; override;

    procedure AddEmitter(Emitter: TParticleEmitter);

    procedure ProcessFrame;

    property Particles: DArray read FPArticles;
  end;
</code></pre>

<p>For each &#8216;game loop&#8217;, the particle manager will loop through all particles, destroying those that have lived their age, and telling the others to update them self. When calling Update( ) on the particle, it pass in the tickcount since update. This is to ensure a relatively smooth and natural movement even if the fps is not stable.</p>

<pre><code> procedure TParticleManager.ProcessFrame;
  var
    CurrentFrame: int64;
    FrameDelta: int64;
  begin
    CurrentFrame := GetTickCount;
    FrameDelta := CurrentFrame - FLastFrame;

    RemoveDeadParticles;
    EmittNewParticles(FrameDelta);
    ProcessParticles(FrameDelta);

    FLastFrame := CurrentFrame;
  end;
</code></pre>

<p>It will also contain a list of TParticleEmitters. The particle emitter will get it&#8217;s chance to emit new particles once every loop. It is the emitters responsibility to give the new particles a initial position and velocity.</p>

<pre><code> TParticleEmitter = class(TObject)
  public
    constructor Create;
    procedure EmittParticles(TimeSinceLastFrame: int64; Particles: DArray);

    property ParticlesPrTick: extended read FParticlesPrTick write FParticlesPrTick;
  end;
</code></pre>

<p>The rendering is separated into it&#8217;s own class, TRendrer. This class will to all the OpenGL initialization-stuff. It is given a list of objects to render, where each item must implement the IRenderItem interface. In this way, the rendrer do not need to know anything about what it renders. It just setup a place for the rendering to happend, and asks the item to draw it self.</p>

<pre><code> TRendrer = class(TObject)
  public
    constructor Create(AHandle: THandle);
    destructor Destroy; override;

    procedure Render(ItemsToRender: DArray);

    procedure HandleResize(NewSize: TRect);
  end;
</code></pre>

<p>To make the particle able to render it self, I have made a sub class, TVisualParticle, which implements IRenderItem. For now, it just draws a small triangle in the particles color.</p>

<pre><code>procedure TVisualParticle.Render;
begin
  glPushMatrix();
  glTranslatef(Position.x, Position.y, Position.z);

  glBegin(GL_TRIANGLES);
    glColor4f(Color.Red, Color.Green, Color.Blue, Color.Alpha);
    glVertex3f(-0.1,-0.1, 0);
    glVertex3f( 0.1,-0.1, 0);
    glVertex3f( 0.0, 0.1, 0);
  glEnd;

  glPopMatrix;
end;
</code></pre>

<p>From here, there are a couple of things that needs improvements. I need to explore OpenGL a bit more to get control over &#8216;the world&#8217; that the particles lives in. Maybe a camera-model or something is needed. I don&#8217;t know. I also need to look into ways of rendering a little more interesting particles then triangles… And then there are physics – particles should be able to react to gravity and other forces as well. I&#8217;m not sure where I will start though. Time will show…</p>